<div class="MainComp">
  <Topbar ref:topbar on:mobilemenu="handleMobileMenu(event)" on:fontsize="handleFontSize(event)" />
  <Menu ref:menu on:mobilemenu="handleMobileMenu(event)" />

  <div class="doc-container font-{{fontSize}}" ref:docContainer>
    {{{ documentationHTML }}}
  </div>
</div>

<script>
  import Settings from '../herodotus-settings.js';
  import Menu from './menu.component';
  import Topbar from './topbar.component';
  import marked from 'marked';
  import ajax from '@fdaciuk/ajax';
  import Prism from 'prismjs';

  export default {
    oncreate () {
      this.registerDocumentation();
    },

    data () {
      return {
        fontSize: 2,
        documentation: '',
        documentationHTML: ''
      }
    },

    components: {
      Menu,
      Topbar
    },

    methods: {
      handleFontSize (event) {
        let action = event.action;
        switch (action) {
          case 'INCREASE':
          this.increaseFont();
          break;
          case 'DECREASE':
          this.decreaseFont();
          break;
        }
      },
      handleMobileMenu (event) {
        let action = event.action;
        switch (action) {
          case 'TOGGLE':
          this.refs.topbar.set({isMenuOpen: this.refs.menu.toggle()});
          break;
          case 'CLOSE':
          this.refs.menu.close();
          break;
          case 'OPEN':
          this.refs.menu.open();
          this.refs.topbar.set({isMenuOpen: true});
          break;
        }
      },
      increaseFont () {
        let fontSize = this.get('fontSize');
        if (fontSize >= 4) {
          return;
        }
        this.set({fontSize: fontSize + 1});
      },
      decreaseFont () {
        let fontSize = this.get('fontSize');
        if (fontSize <= 1) {
          return;
        }
        this.set({fontSize: fontSize - 1});
      },
      registerDocumentation () {
        let DOC_URL;
        
        if(Settings.allow_remote_documentation) {
          DOC_URL = Settings.remote_documentation_url;
        } else {
          DOC_URL = 'documentation.md';
        }

        ajax().get(DOC_URL).then((res, xhr) => {
          // Print documentation HTML
          this.set({
            documentation: res,
            documentationHTML: marked(res)
          });

          // Send menu to menu component
          this.refs.menu.startMenu(this.generateMenu());

          // Highlight code
          Prism.highlightAll();
        });
      },
      generateMenu () {
        let docElements = this.refs.docContainer;
        let headingItems = docElements.querySelectorAll('h1, h2');
        let menuItems = [];

        headingItems.forEach((item) => {
          menuItems.push({
            type: item.nodeName.toLowerCase(),
            label: item.textContent,
            id: item.id
          });
        });

        return menuItems;
      }
    }
  }
</script>