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

    <div v-else @click="aabenEksterntLink" :class="['aula-widget-wrapper', widgetStorrelse]">
      <h2>Nyt fra {{ portalTitel }}</h2>
      
      <div class="indhold-wrapper">
        <div v-if="billedeUrl" class="billede-ramme">
          <img :src="billedeUrl" alt="Forløbsbillede" />
        </div>
        <h1>{{ labelTekst }}</h1>
      </div>
      
      <div v-if="widgetStorrelse !== 'minimal'" class="body-scroll-container">
        <widget-html v-if="bodyTekst" :html="bodyTekst" class="brødtekst"></widget-html>
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

    // Det pæne navn til overskriften (fx "Læringsportalen Skive")
    portalTitel() {
      return this.aktuelConfig ? (this.aktuelConfig.title || this.aktuelConfig.domain) : '';
    },

    // Det rene domæne til tekniske stier (fx "laeringsportalenskive.dk")
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
    },

    // Returnerer enten 'minimal', 'medium' eller 'large'
    widgetStorrelse() {
      return this.aktuelConfig ? (this.aktuelConfig.widgetSize || 'medium') : 'medium';
    },

    antalTilbud() {
      return this.aktuelConfig ? (this.aktuelConfig.numberOfOffers || 1) : 1;
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
        const configUrl = `https://api.iklog.dk/beta/iklogudoglaer.php?kode=${this.aktueltInstNr}`;
        
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

        // Gemmer de returnerede indstillinger fra PHP
        this.aktuelConfig = {
          domain: configData.domain,
          path: configData.path,
          title: configData.title || configData.domain,
          widgetSize: configData.widgetSize || 'medium',
          numberOfOffers: configData.numberOfOffers || 1
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

      // 1. Send klik-registrering via ren URL og custom HTTP-header (omgår adblockere)
      const clickUrl = `https://api.iklog.dk/beta/iklogudoglaer.php?kode=${this.aktueltInstNr}`;

      fetch(clickUrl, {
        method: 'GET',
        headers: {
          'X-Widget-Action': 'open'
        },
        keepalive: true
      }).catch(() => {});

      // 2. Udfør godkendt navigation via skema-submit
      const form = document.getElementById('aula-secure-navigation-form');
      if (form) {
        setTimeout(() => {
          form.submit();
        }, 80);
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
/* Standard container opsætning */
.aula-widget-wrapper {
  position: relative;
  box-sizing: border-box;
  padding: 10px;
  font-family: Arial, sans-serif;
  cursor: pointer;
  container-type: inline-size;
}

/* Standard opstilling (large): Billede øverst, overskrift under */
.indhold-wrapper {
  display: block;
}

/* Standard scroll-container til brødtekst (large) */
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

/* Tekst-skalering og tekstbrydning på overskriften */
h2 { 
  font-size: clamp(10px, 4cqw, 14px); 
  color: #666; 
  margin: 0 0 5px 0; 
  text-transform: uppercase; 
  letter-spacing: 0.5px;
  line-height: 1.2;
  overflow-wrap: break-word;
  word-break: break-word;
}

h1 { 
  font-size: 18px; 
  margin-top: 0; 
  margin-bottom: 10px; 
  color: #000; 
  line-height: 1.2; 
  overflow-wrap: break-word;
}

.billede-ramme { width: 100%; margin-bottom: 10px; }
img { width: 100%; height: auto; display: block; }
.brødtekst { font-size: 14px; line-height: 1.4; color: #333; padding-bottom: 25px; }

.floating-tekst-fixed {
  position: sticky;
  bottom: 0;
  left: 0;
  width: 100%;
  background: linear-gradient(to top, rgba(255,255,255,0.95) 70%, rgba(255,255,255,0) 100%);
  color: #0056b3;
  font-size: 11px;
  font-weight: bold;
  text-align: center;
  padding-top: 5px;
  pointer-events: none;
}

/* -------------------------------------------------------------
   MINIMAL & MEDIUM: Billede fylder 30% side om side med titel
   ------------------------------------------------------------- */
.aula-widget-wrapper.minimal .indhold-wrapper,
.aula-widget-wrapper.medium .indhold-wrapper {
  display: flex;
  align-items: center;
  gap: 10px;
}

.aula-widget-wrapper.minimal .billede-ramme,
.aula-widget-wrapper.medium .billede-ramme {
  flex: 0 0 30%;
  width: 30%;
  margin-bottom: 0;
}

.aula-widget-wrapper.minimal .billede-ramme img,
.aula-widget-wrapper.medium .billede-ramme img {
  width: 100%;
  height: auto;
  border-radius: 4px;
  object-fit: cover;
}

.aula-widget-wrapper.minimal h1,
.aula-widget-wrapper.medium h1 {
  flex: 1;
  font-size: 14px;
  margin: 0;
  line-height: 1.3;
}

/* -------------------------------------------------------------
   MEDIUM SPECIFIK: Tilpasset højde på brødtekstboksen
   ------------------------------------------------------------- */
.aula-widget-wrapper.medium .body-scroll-container {
  height: 100px;
  margin-top: 8px;
}
</style>