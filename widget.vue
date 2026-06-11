<template>
  <div>
    <form 
      id="aula-secure-navigation-form" 
      :action="fuldEksternUrl" 
      method="GET" 
      target="_blank" 
      style="display: none;">
    </form>

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

    <div v-else @click="aabenEksterntLink" class="aula-widget-wrapper">
      <h2>Nyt fra {{ siteDomain }}</h2>
      <h1>{{ labelTekst }}</h1>
      
      <div v-if="billedeUrl" class="billede-ramme">
        <img :src="billedeUrl" alt="Forløbsbillede" />
      </div>
      
      <div class="body-scroll-container">
        <widget-html :html="bodyTekst" class="brødtekst"></widget-html>
        <div class="floating-tekst-fixed">Klik for at læse mere...</div>
      </div>
    </div>
  </div>
</template>

<script>
module.exports = {
  props: ['placement', 'institutionFilter', 'aulaUuid', 'aulaToken'],
  
  data() {
    return {
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
    aktueltInstNr() {
      if (!this.institutionFilter) return null;
      return Array.isArray(this.institutionFilter) 
        ? String(this.institutionFilter[0]).trim() 
        : String(this.institutionFilter).trim();
    },

    siteDomain() {
      return this.aktuelConfig ? this.aktuelConfig.domain : '';
    },

    apiPath() {
      return this.aktuelConfig ? this.aktuelConfig.path : '';
    },

    apiBaseUrl() { 
      return this.siteDomain ? `https://api.${this.siteDomain}` : ''; 
    },
    
    wwwBaseUrl() { 
      if (!this.siteDomain) return '';
      return ['laeringsportalenskive.dk', 'ulfiaarhus.dk', 'ude.nu'].includes(this.siteDomain)
        ? `https://www.${this.siteDomain}` 
        : `https://${this.siteDomain}`; 
    },
    
    apiUrl() { 
      if (!this.apiBaseUrl || !this.apiPath) return '';
      return `${this.apiBaseUrl}${this.apiPath}?format=json&region=content&sort_by=created`; 
    },

    fuldEksternUrl() {
      if (!this.forloebsLink || !this.wwwBaseUrl) return '';
      return `${this.wwwBaseUrl}${this.forloebsLink}`;
    }
  },

  methods: {
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

    async hentData() {
      if (!this.aktueltInstNr) {
        this.ikkeUnderstoettet = true;
        this.loading = false;
        return;
      }

      try {
        this.status = 'Henter konfiguration...';
        const configUrl = `https://api.iklog.dk/iklogudoglaer.php?kode=${this.aktueltInstNr}`;
        
        const headers = {};
        if (this.aulaUuid) {
          headers['X-Aula-Session-UUID'] = this.aulaUuid;
        }
        
        const configRes = await this.fetchMedTimeout(configUrl, { headers });
        if (!configRes.ok) throw new Error('Kunne ikke hente konfiguration fra serveren');
        const configData = await configRes.json();
        
        if (configData.status !== 'success' || !configData.domain || !configData.path) {
          this.ikkeUnderstoettet = true;
          this.loading = false;
          return;
        }

        this.aktuelConfig = {
          domain: configData.domain,
          path: configData.path
        };

        this.status = 'Indlæser indhold...';
        
        if (!this.apiUrl) {
          throw new Error('Konfigurationsstier ikke klar.');
        }

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
        
        const res2 = await this.fetchMedTimeout(`${this.apiBaseUrl}${udtrukketLink}?format=json`);
        if (!res2.ok) throw new Error('Netværksfejl ved hentning af indhold');
        const data2 = await res2.json();
        
        const indhold = data2?.content?.content;
        
        if (indhold) {
          this.labelTekst = indhold.label || 'Ingen titel fundet';
          this.bodyTekst = indhold.body || '';
          this.forloebsLink = indhold.link || '';
          
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
        
      } catch (fejl) {
        this.apiFejlBesked = 'Vi kunne desværre ikke indlæse indholdet i øjeblikket. Prøv igen senere.';
        this.loading = false;
      }
    },
    
    aabenEksterntLink() {
      if (!this.forloebsLink) return;
      
      // Standard DOM-opslag i stedet for det ulovlige this.$refs jf. afsnit 4.8
      const form = document.getElementById('aula-secure-navigation-form');
      if (form) {
        form.submit();
      }
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
/* Alt CSS er isoleret med 'scoped' jf. afsnit 4.1.2 og 4.8 */
.aula-widget-wrapper {
  position: relative;
  box-sizing: border-box;
  padding: 10px;
  font-family: Arial, sans-serif;
  cursor: pointer;
}

/* Scroll-container tager nu også fuld dynamisk bredde */
.body-scroll-container {
  position: relative;
  height: 150px; 
  overflow-y: auto;
  box-sizing: border-box;
  margin-top: 5px;
}

.loading-tekst {
  font-family: Arial, sans-serif;
  font-size: 14px;
  padding: 15px;
  color: #333;
}

.ikke-understoettet-boks, .api-fejl-boks {
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
.billede-ramme { width: 100%; margin-bottom: 10px; }
img { width: 100%; height: auto; display: block; }
.brødtekst { font-size: 14px; line-height: 1.4; color: #333; padding-bottom: 20px; }

.floating-tekst-fixed {
  position: sticky;
  bottom: 0;
  left: 0;
  width: 100%;
  background: linear-gradient(to top, rgba(255,255,255,0.9) 60%, rgba(255,255,255,0) 100%);
  color: #0056b3;
  font-size: 11px;
  font-weight: bold;
  text-align: center;
  padding-top: 5px;
  pointer-events: none;
}
</style>