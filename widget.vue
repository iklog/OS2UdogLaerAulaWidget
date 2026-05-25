<script>
// Dit samlede institutions-objekt
const INSTITUTIONS = {
  thisted_skoler: ['280215','280220','280811','281197','281396','765001','765004','765005','785002','785006','785007','785009','785013','785015','785016','785301','787002','787004','787005','787006','787008','787012','787013','787016','787019','787021','787022','787024','787030','787033','787035','787036','787214','787303','787000'],
  thisted_dagtilbud: ['G21469','G21673','G21674','G21675','G21676','G21677','G21678','G21704','G21705','G21706','G21707','G21708','G21731','G21775','G22746','G22747','G22748','G22749','G22750','G22751','G22752','G22753','G22754','G22755','G22756','G22757','G22759','G22760','G22761','G22762','G22763','G22764','G22765','G22768'],
  aarhus_skoler: ['280114','280144','280146','280307','280313','280397','280410','280458','281191','281290','281401','281403','281404','281520','281602','281648','281689','751002','751003','751004','751006','751007','751008','751013','751014','751015','751016','751017','751018','751019','751020','751021','751022','751023','751024','751025','751026','751027','751032','751033','751034','751035','751036','751038','751039','751040','751041','751042','751043','751044','751045','751046','751050','751051','751052','751053','751054','751055','751056','751058','751059','751060','751061','751062','751063','751064','751065','751066','751080','751081','751089','751090','751094','751095','751098','751107','751111','751114','751204','751312','751327','751903','791413'],    
  aarhus_dagtilbud: ['G19082','G19083','G19084','G19085','G19086','G19087','G19088','G19089','G19090','G19091','G19092','G19093','G19094','G19095','G19096','G19097','G19098','G19099','G19100','G19101','G19102','G19103','G19104','G19105','G19106','G19107','G19108','G19109','G19110','G19111','G19112','G19113','G19114','G19115','G19116','G19117','G19118','G19119','G19120','G19121','G19122','G19123','G19124','G19125','G19126','G19127','G19128','G19129','G19130','G19131','G19132','G19133','G19134','G19135','G19136','G19137','G19138','G19139','G19140','G19141','G19142','G19143','G19144','G19145','G19146','G19147','G19148','G19149','G19150','G19151','G19152','G19153','G19154','G19155','G19156','G19157','G19158','G19159','G19160','G19161','G19162','G19163','G19164','G19165','G19166','G19167','G19168','G19170','G19171','G19172','G19173','G19174','G19175','G19176','G19177','G19178','G19179','G19180','G19181','G19182','G19183'],
  aalborg_skoler: ['281767', '817001', '817002', '817004', '817005', '831002', '831003', '831006', '837002', '837003', '837005', '837006', '837007', '851005', '851006', '851007', '851009', '851010', '851011', '851014', '851015', '851017', '851019', '851020', '851021', '851022', '851024', '851026', '851027', '851028', '851029', '851030', '851031', '851032', '851034', '851036', '851037', '851039', '851040', '851041', '851042', '851044', '851049', '851051', '851052', '851053', '851054', '851055', '851064', '851066', '851068', '851070', '851071', '851076', '851087'],
  esbjerg_skoler: ['280205', '280500', '280501', '280502', '282147', '282239', '282240', '282241', '282242', '282243', '282244', '282245', '282246', '282247', '282248', '282249', '282250', '282251', '282252', '557001', '557002', '557003', '557005', '557006', '557007', '561002', '561003', '561004', '561005', '561006', '561011', '561014', '561015', '561016', '561018', '561019', '561022', '561024', '561026', '561030', '561036', '561039', '571002', '571015', '571017', '571022'],
  randers_skoler: ['280324', '280455', '280708', '280741', '281140', '281195', '281223', '281313', '281395', '281397', '281488', '281534', '281615', '281656', '711003', '711004', '711006', '711007', '711011', '711013', '711014', '711015', '711016', '730001', '730003', '730004', '730005', '730006', '730007', '730010', '730011', '730012', '730014', '730017', '730018', '730022', '730026', '730030', '730032', '730036', '730037', '730038', '730040', '730041', '730042', '730043', '730044', '730045', '730048', '730052', '730054', '730056', '730062', '730068', '730070', '730072'],
  skive_skoler: ['280534', '281006', '281779', '281780', '281781', '281787', '281806', '777001', '777002', '777004', '779001', '779003', '779004', '779005', '779006', '779007', '779008', '779009', '779011', '779015', '779017', '781005', '781006', '783005'],
  vejle_skoler: ['280331', '280332', '280333', '280334', '280335', '280463', '280474', '280537', '281635', '281813', '281814', '281817', '281818', '281819', '281820', '281821', '281822', '603002', '603004', '603005', '605002', '605007', '605008', '605009', '611008', '611010', '611011', '611013', '617001', '617002', '627008', '631002', '631003', '631005', '631006', '631009', '631011', '631012', '631013', '631014', '631015', '631017', '631018', '631019', '631023', '631024', '631029', '631034'],
  viborg_skoler: ['280384', '281294', '281584', '281892', '281893', '282151', '761006', '761011', '761012', '761016', '763001', '763005', '763006', '763008', '769001', '769003', '775001', '775004', '775006', '789002', '789005', '789008', '789009', '791002', '791003', '791004', '791005', '791006', '791008', '791010', '791013', '791014', '791015', '791019', '791020', '791027', '791028', '791029'],
  
  aalborg_dagtilbud: ['G11078', 'G11080', 'G15689', 'G15691', 'G15701', 'G15703', 'G15705', 'G15711', 'G15713', 'G15715', 'G15717', 'G15719', 'G15721', 'G15723', 'G15725', 'G15727', 'G15729', 'G15731', 'G15733', 'G15737', 'G15739', 'G15741', 'G15743', 'G15745', 'G15747', 'G15749', 'G15751', 'G15756', 'G15760', 'G15762', 'G15764', 'G15768', 'G15770', 'G15772', 'G15774', 'G15778', 'G15770', 'G15772', 'G15774', 'G15778', 'G15780', 'G15782', 'G15784', 'G15786', 'G15788', 'G15790', 'G15794', 'G15796', 'G15798', 'G15800', 'G15802', 'G15804', 'G15806', 'G15808', 'G15810', 'G15812', 'G15814', 'G15818', 'G15820', 'G15822', 'G15824', 'G15826', 'G15828', 'G15820', 'G15822', 'G15824', 'G15826', 'G15828', 'G15830', 'G15832', 'G15834', 'G15836', 'G15838', 'G15840', 'G15842', 'G15846', 'G15848', 'G15850', 'G15852', 'G15854', 'G15856', 'G15858', 'G15860', 'G15862', 'G15864', 'G15866', 'G15868', 'G15870', 'G15872', 'G15874', 'G15876', 'G15878', 'G15880', 'G15882', 'G15884', 'G15886', 'G15888', 'G15890', 'G15892', 'G15894', 'G15898', 'G15900', 'G15902', 'G15904', 'G15906', 'G15908', 'G15910', 'G15912', 'G15914', 'G15916', 'G15918', 'G15920', 'G15922', 'G15924', 'G15926', 'G15930', 'G15932', 'G15934', 'G15936', 'G15938', 'G15940', 'G15942', 'G15944', 'G15946', 'G15948', 'G15950', 'G15952', 'G15954', 'G15956', 'G15958', 'G15960', 'G15962', 'G15964', 'G15966', 'G15968', 'G15970', 'G15972', 'G15974', 'G15978', 'G15980', 'G15982', 'G15984', 'G15986', 'G15988', 'G15994', 'G15996', 'G15998', 'G16000', 'G16167', 'G16330', 'G16332', 'G16334', 'G16336', 'G16594', 'G16596', 'G16598', 'G16600', 'G16602', 'G16604', 'G16606', 'G16608', 'G16610', 'G16612', 'G17290', 'G17291', 'G17292', 'G17293', 'G17514', 'G17515', 'G17516', 'G17517', 'G17518', 'G17520', 'G17522', 'G17524', 'G17527', 'G17529', 'G17531', 'G17535', 'G17725', 'G23494', 'G23499', 'G23562', 'G23608', 'G24215', 'G24313', 'G24319', 'G24393', 'G24501', 'G24607', 'G24711', 'G25036', 'G25180', 'G25196', 'G25334', 'G25525', 'G25542', 'G25543', 'G25544', 'G25545', 'G25559', 'G25560', 'G25561', 'G25562', 'G25563', 'G25564', 'G25565', 'G25566', 'G25567', 'G25582', 'G25739', 'G25961', 'G27121'],
  esbjerg_dagtilbud: ['G13785', 'G13787', 'G13789', 'G13791', 'G13793', 'G13795', 'G13797', 'G13799', 'G13801', 'G13805', 'G13807', 'G13809', 'G13811', 'G13813', 'G13815', 'G13817', 'G13819', 'G13821', 'G13822', 'G13825', 'G13826', 'G13828', 'G13820', 'G13822', 'G13825', 'G13826', 'G13828', 'G13832', 'G13833', 'G13834', 'G13836', 'G13837', 'G13838', 'G13839', 'G13840', 'G13843', 'G13844', 'G13845', 'G13846', 'G13848', 'G13849', 'G13851', 'G13852', 'G13854', 'G13855', 'G13857', 'G13859', 'G13860', 'G13862', 'G13863', 'G13864', 'G17676', 'G17677', 'G17678', 'G17679', 'G22664', 'G24905', 'G25202', 'G25206', 'G25207', 'G25208', 'G25211', 'G25213', 'G25214', 'G25216', 'G25984', 'G30357', 'G30358'],
  randers_dagtilbud: ['G19806', 'G19808', 'G19809', 'G19810', 'G19811', 'G19812', 'G19813', 'G19814', 'G19815', 'G19816', 'G19817', 'G19818', 'G19819', 'G19820', 'G19821', 'G19822', 'G20089', 'G20090', 'G20091', 'G20092', 'G20093', 'G20094', 'G20095', 'G20096', 'G20097', 'G20098', 'G20099', 'G20100', 'G20101', 'G20102', 'G20103', 'G20104', 'G20307', 'G20308', 'G20311', 'G20312', 'G20313', 'G20314', 'G20315', 'G20316', 'G20317', 'G20318', 'G20319', 'G20405', 'G20406', 'G20407', 'G20408', 'G20409', 'G20452', 'G20453', 'G20500', 'G20502', 'G20505', 'G20524', 'G20526', 'G20527', 'G20528', 'G20537', 'G20538', 'G20539', 'G20541', 'G20542', 'G20543', 'G20549', 'G20550', 'G23655', 'G24257', 'G24304', 'G24305', 'G24333', 'G24397', 'G24439', 'G24489', 'G24715', 'G24869', 'G24913', 'G25164', 'G25183', 'G25628', 'G25965', 'G27151', 'G27152'],
  skive_dagtilbud: ['G19962', 'G19963', 'G19964', 'G19966', 'G19967', 'G19968', 'G19969', 'G19970', 'G19971', 'G19972', 'G19973', 'G19974', 'G19975', 'G19976', 'G19977', 'G19978', 'G19979', 'G19980', 'G19981', 'G19982', 'G19983', 'G19984', 'G20238', 'G20239', 'G20240', 'G20241', 'G20242', 'G20243', 'G20246', 'G20247', 'G20380', 'G20381', 'G20382', 'G20383', 'G20384', 'G20385', 'G20445', 'G20447', 'G20487', 'G20492', 'G20493', 'G20494', 'G24195', 'G24659', 'G24695', 'G24823', 'G25089'],
  vejle_dagtilbud: ['G14307', 'G14309', 'G14315', 'G14317', 'G14319', 'G14321', 'G14323', 'G14325', 'G16386', 'G19729', 'G19730', 'G19732', 'G19734', 'G19736', 'G19737', 'G19738', 'G20048', 'G20050', 'G20051', 'G20052', 'G20282', 'G20284', 'G20285', 'G20396', 'G20397', 'G20682', 'G20700', 'G20715', 'G20717', 'G20742', 'G20743', 'G20744', 'G20745', 'G20746', 'G20781', 'G20782', 'G20783', 'G20818', 'G20819', 'G20820', 'G20849', 'G20851', 'G20852', 'G20892', 'G20893', 'G20894', 'G20895', 'G20897', 'G20898', 'G20899', 'G20900', 'G20941', 'G20942', 'G20943', 'G20944', 'G20945', 'G20946', 'G20963', 'G20965', 'G20997', 'G20998', 'G21041', 'G23613', 'G24151', 'G24327', 'G24509', 'G24765', 'G25295', 'G25297', 'G25487', 'G26378', 'G26594', 'G27144'],
  viborg_dagtilbud: ['G15382', 'G15384', 'G15386', 'G15388', 'G15390', 'G15392', 'G15394', 'G15396', 'G15396', 'G15398', 'G15400', 'G15417', 'G15425', 'G16091', 'G16691', 'G16693', 'G16695', 'G16697', 'G16698', 'G16699', 'G16700', 'G16703', 'G16708', 'G16710', 'G16711', 'G16713', 'G16714', 'G16716', 'G16718', 'G16719', 'G16720', 'G16722', 'G16723', 'G16728', 'G16731', 'G16732', 'G16733', 'G16734', 'G16737', 'G16739', 'G16740', 'G16742', 'G16743', 'G16747', 'G16750', 'G16752', 'G16753', 'G16754', 'G16755', 'G16756', 'G16760', 'G16761', 'G16762', 'G16763', 'G16765', 'G23630', 'G24395', 'G24767']
};

// Konfiguration der parrer gruppenavne med deres specifikke opsætning
const GROUP_CONFIG = {
  thisted_skoler: { domain: 'rumthisted.dk', path: '/grundskole-3' },
  thisted_dagtilbud: { domain: 'rumthisted.dk', path: '/dagtilbud-5' },
  aarhus_skoler: { domain: 'ulfiaarhus.dk', path: '/grundskole-0' },
  aarhus_dagtilbud: { domain: 'ulfiaarhus.dk', path: '/dagtilbud-0' },
  aalborg_skoler: { domain: 'aabenaalborg.dk', path: '/grundskole-1' },
  aalborg_dagtilbud: { domain: 'aabenaalborg.dk', path: '/dagtilbud-4' },
  esbjerg_skoler: { domain: 'ude.nu', path: '/skole' },
  esbjerg_dagtilbud: { domain: 'ude.nu', path: '/dagtilbud-1' },
  randers_skoler: { domain: 'udoglaer.randers.dk', path: '/grundskole-2' },
  randers_dagtilbud: { domain: 'udoglaer.randers.dk', path: '/dagtilbud-3' },
  skive_skoler: { domain: 'laeringsportalenskive.dk', path: '/aaben-skole' },
  skive_dagtilbud: { domain: 'laeringsportalenskive.dk', path: '/aaben-dagtilbud' },
  vejle_skoler: { domain: 'udoglaer.vejle.dk', path: '/grundskole-4' },
  vejle_dagtilbud: { domain: 'udoglaer.vejle.dk', path: '/dagtilbud' },
  viborg_skoler: { domain: 'klcviborg.dk', path: '/grundskole' },
  viborg_dagtilbud: { domain: 'klcviborg.dk', path: '/dagtilbud-2' }
};

// BYGGER DATASTRUKTUREN ÉN GANG
const INST_MAP = new Map();
for (const [groupName, ids] of Object.entries(INSTITUTIONS)) {
  const config = GROUP_CONFIG[groupName];
  if (config) {
    ids.forEach(id => INST_MAP.set(id, config));
  }
}

module.exports = {
  props: ['setIframeHeight', 'placement', 'institutionFilter'],
  
  data() {
    return {
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

    // O(1) opslag
    aktuelConfig() {
      return INST_MAP.get(this.aktueltInstNr) || null;
    },

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
      return this.siteDomain === 'laeringsportalenskive.dk'
        || this.siteDomain === 'ulfiaarhus.dk'
        || this.siteDomain === 'ude.nu' 
        ? `https://www.${this.siteDomain}` 
        : `https://${this.siteDomain}`; 
    },
    
    apiUrl() { 
      return `${this.apiBaseUrl}${this.apiPath}?format=json&region=content&sort_by=created`; 
    }
  },

  methods: {
    setHeight(height) {
      if (this.setIframeHeight) {
        this.setIframeHeight(height);
      } else {
        // Fallback for native Aula Iframe PostMessage (T0150 Widget guide)
        window.parent.postMessage({
          request: 'setIframeHeight',
          metadata: { height: height }
        }, '*'); // Overvej at skifte '*' til specifik Aula origin i prod?
      }
    },

    async fetchMedTimeout(url, options = {}) {
      const timeout = 8000; // 8 sekunders timeout
      const controller = new AbortController();
      const id = setTimeout(() => controller.abort(), timeout);
      
      const response = await fetch(url, { ...options, signal: controller.signal });
      clearTimeout(id);
      return response;
    },

    async hentData() {
      if (!this.aktuelConfig) {
        this.ikkeUnderstoettet = true;
        this.loading = false;
        this.setHeight(260);
        return;
      }

      try {
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
                if (source?.srcset) bedsteBillede = source.srcset.split(' ')[0];
              }
            }
          });
          
          this.billedeUrl = bedsteBillede || indhold.field_image?.img_element?.uri || '';
        }
        
        this.loading = false;
        this.setHeight(350);
        
      } catch (fejl) {
        // Kun generel besked ud til pædagogerne i produktion. 
        // Den rå fejl kan opsamles i et logging tool som Sentry hvis muligt.
        this.apiFejlBesked = 'Vi kunne desværre ikke indlæse indholdet i øjeblikket. Prøv igen senere.';
        this.loading = false;
        this.setHeight(150); 
      }
    },
    
    aabenEksterntLink() {
      if (this.forloebsLink) {
        console.log(`${this.wwwBaseUrl}${this.forloebsLink}`);
        window.open(`${this.wwwBaseUrl}${this.forloebsLink}`, '_blank');
      }
    }
  },

  mounted() {
    this.hentData();
  }
}
</script>

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

<style scoped>
.aula-widget-scroll {
  position: relative;
  width: 100%; /* Sørger for den tilpasser sig iframe-bredden som defineret af Aula */
  max-width: 300px;
  height: 350px;
  max-height: 350px;
  overflow-y: auto;
  cursor: pointer;
  box-sizing: border-box;
  padding: 10px;
  font-family: Arial, sans-serif; /* Tilpasset Aulas fallback i designguiden */
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
  color: #333; /* Mere neutral og tilpasset Aulas standardfarver */
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