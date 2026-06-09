<template>
  <div>
    <div v-if="apiFejlBesked" class="api-fejl-boks">
      <p>{{ apiFejlBesked }}</p>
    </div>

    <p v-else-if="loading" class="loading-tekst">
      <i class="fa fa-spinner fa-spin"></i> {{ status }}
    </p>
    
    <div v-else-if="ikkeUnderstoettet" class="ikke-understoettet-boks">
      <p>Din institution understøtter i øjeblikket ikke denne widget.</p>
      <p>Hvis din kommune er med i OS2udoglær, kan funktionen dog nemt aktiveres fremover.</p>
    </div>

    <div v-else @click="aabenEksterntLink" class="aula-widget-scroll">
      <h2>Nyt fra {{ siteDomain }}</h2>
      <h1>{{ labelTekst }}</h1>
      
      <div v-if="billedeUrl">
        <img :src="billedeUrl" alt="Forløbsbillede" />
      </div>
      
      <widget-html :html="bodyTekst" class="brødtekst"></widget-html>
      <div class="floating-tekst-fixed">Klik for at læse mere...</div>
    </div>
  </div>
</template>

<script>
module.exports = {
  props: ['setIframeHeight', 'placement', 'institutionFilter'],
  
  data() {
    return {
      // Modtager objektet med { domain, path } dynamisk fra iklog.dk
      aktuelConfig: null,
      
      labelTekst: '',
      bodyTekst: '',
      billedeUrl: '',
      forloebsLink: '',
      loading: true,
      status: 'Indlæser...',
      ikkeUnderstoettet: false,
      apiFejlBesked: ''
    }
  },

  computed: {
    // Renser og formaterer institutionsnummeret fra Aula
    aktueltInstNr() {
      if (!this.institutionFilter) return null;
      return Array.isArray(this.institutionFilter) 
        ? String(this.institutionFilter[0]).trim() 
        : String(this.institutionFilter).trim();
    },

    // De efterfølgende computed properties reagerer proaktivt så snart 'this.aktuelConfig' ændrer sig
    siteDomain() {
      return this.aktuelConfig ? this.aktuelConfig.domain : 'rumthisted.dk';
    },

    apiPath() {
      return this.aktuelConfig ? this.aktuelConfig.path : '/grundskole-3';
    },

    apiBaseUrl() { 
      return `https://api.${this.siteDomain}`; 
    },
    
    wwwBaseUrl() { 
      return ['laeringsportalenskive.dk', 'ulfiaarhus.dk', 'ude.nu'].includes(this.siteDomain)
        ? `https://www.${this.siteDomain}` 
        : `https://${this.siteDomain}`; 
    },
    
    apiUrl() { 
      return `${this.apiBaseUrl}${this.apiPath}?format=json&region=content&sort_by=created`; 
    }
  },

  methods: {
    // Sørger for at iFramen tilpasser sig i højden inde i Aula
    setHeight(height) {
      if (this.setIframeHeight) {
        this.setIframeHeight(height);
      } else {
        window.parent.postMessage({
          request: 'setIframeHeight',
          metadata: { height: height }
        }, '*'); // Erstat evt. '*' med Aulas specifikke widget-domain i prod for optimal sikkerhed
      }
    },

    // Generisk fetch-hjælper med indbygget timeout på 8 sekunder
    async fetchMedTimeout(url, options = {}) {
      const timeout = 8000;
      const controller = new AbortController();
      const id = setTimeout(() => controller.abort(), timeout);
      
      try {
        const response = await fetch(url, { ...options, signal: controller.signal });
        clearTimeout(id);
        return response;
      } catch (e) {
        clearTimeout(id);
        throw e;
      }
    },

    // Hovedmetoden der orkestrerer både config-hentning og Drupal-indholdshentning
    async hentData() {
      if (!this.aktueltInstNr) {
        this.ikkeUnderstoettet = true;
        this.loading = false;
        this.setHeight(260);
        return;
      }

      try {
        // TRIN 1: Hent portalens opsætning dynamisk baseret på institutionsnummeret
        this.status = 'Henter konfiguration...';
        const configUrl = `https://iklog.dk/widget/index.php?kode=${this.aktueltInstNr}`;
        const configRes = await this.fetchMedTimeout(configUrl);
        
        if (!configRes.ok) throw new Error('Kunne ikke hente konfiguration fra serveren');
        const configData = await configRes.json();
        
        // Hvis databasen ikke kender koden eller mangler data, vises "ikke understøttet" skærmen
        if (configData.status !== 'success' || !configData.domain || !configData.path) {
          this.ikkeUnderstoettet = true;
          this.loading = false;
          this.setHeight(260);
          return;
        }

        // Gemmer data lokalt, hvilket øjeblikkeligt opdaterer alle computed URL-stier i baggrunden
        this.aktuelConfig = {
          domain: configData.domain,
          path: configData.path
        };

        // TRIN 2: Hent layoutet fra den specifikke kommunes portal
        this.status = 'Indlæser indhold...';
        const res1 = await this.fetchMedTimeout(this.apiUrl);
        if (!res1.ok) throw new Error('Netværksfejl ved hentning af layout');
        const data1 = await res1.json();
        
        const layoutRows = data1?.content?.layout_builder__layout || [];
        let udtrukketLink = layoutRows[0]?.regions?.content?.[0]?.results?.[0]?.link;
        
        if (!udtrukketLink) {
          const viewSection = layoutRows.find(row => row?.regions?.content?.some(item => item?.type === 'view'));
          const viewModule = viewSection?.regions?.content?.find(item => item?.type === 'view');
          udtrukketLink = viewModule?.results?.[0]?.link;
        }
        
        if (!udtrukketLink) {
          throw new Error('Der er ikke lagt noget aktivt forløb på denne portal endnu.');
        }
        
        // TRIN 3: Hent det specifikke forløbsdata (Brødtekst, titel og billeder)
        const res2 = await this.fetchMedTimeout(`${this.apiBaseUrl}${udtrukketLink}?format=json`);
        if (!res2.ok) throw new Error('Netværksfejl ved hentning af indhold');
        const data2 = await res2.json();
        
        const indhold = data2?.content?.content;
        
        if (indhold) {
          this.labelTekst = indhold.label || 'Ingen titel fundet';
          this.bodyTekst = indhold.body || '';
          this.forloebsLink = indhold.link || '';
          
          // Find det billede fra Drupal der matcher de 300px bedst
          const sources = indhold.field_image?.sources || [];
          let bedsteBillede = null;
          let mindsteForskel = Infinity;
          
          sources.forEach(source => {
            const bredde = parseInt(source?.width, 10);
            if (!isNaN(bredde)) {
              const forskel = Math.abs(bredde - 300);
              if (forskel < mindsteForskel) {
                mindsteForskel = forskel;
                if (source?.srcset) bedsteBillede = source.srcset.trim().split(/\s+/)[0];
              }
            }
          });
          
          this.billedeUrl = bedsteBillede || indhold.field_image?.img_element?.uri || '';
        }
        
        this.loading = false;
        this.setHeight(350);
        
      } catch (fejl) {
        this.apiFejlBesked = 'Vi kunne desværre ikke indlæse indholdet i øjeblikket. Prøv igen senere.';
        this.loading = false;
        this.setHeight(150); 
      }
    },
    
    // Håndterer klik på widgetten, så det eksterne link åbnes i en ny fane
    aabenEksterntLink() {
      // Forhindrer fejl/blanke sider hvis brugeren klikker før data er færdighentet
      if (!this.forloebsLink) return;
      window.open(`${this.wwwBaseUrl}${this.forloebsLink}`, '_blank');
    }
  },

  mounted() {
    this.hentData().catch(err => {
      console.error('Kritisk fejl under widget initialisering:', err);
    });
  }
}
</script>

<style scoped>
.aula-widget-scroll {
  position: relative;
  width: 100%;
  max-width: 300px;
  height: 350px;
  max-height: 350px;
  overflow-y: auto;
  cursor: pointer;
  box-sizing: border-box;
  padding: 10px;
  font-family: Arial, sans-serif;
}

.loading-tekst {
  font-family: Arial, sans-serif;
  font-size: 14px;
  padding: 15px;
  color: #333;
}

.ikke-understoettet-boks, .api-fejl-boks {
  width: 100%;
  max-width: 300px;
  box-sizing: border-box;
  padding: 15px;
  font-family: Arial, sans-serif;
  font-size: 13px;
  line-height: 1.4;
  color: #333;
  background-color: #f9f9f9;
  border-radius: 4px;
  border: 1px solid #ddd;
}

h2 { font-size: 14px; color: #666; margin: 0 0 5px 0; text-transform: uppercase; letter-spacing: 0.5px; }
h1 { font-size: 18px; margin-top: 0; margin-bottom: 10px; color: #000; line-height: 1.2; }
img { width: 100%; height: auto; display: block; margin-bottom: 15px; }
.brødtekst { font-size: 14px; line-height: 1.4; color: #333; padding-bottom: 15px; }

.floating-tekst-fixed {
  position: sticky;
  bottom: 0;
  left: 0;
  width: 100%;
  background: linear-gradient(to top, rgba(255,255,255,0.7) 60%, rgba(255,255,255,0) 100%);
  color: #0056b3;
  font-size: 11px;
  font-weight: bold;
  text-align: center;
  padding: 0;
  margin-top: -11px;
  pointer-events: none;
}
</style>