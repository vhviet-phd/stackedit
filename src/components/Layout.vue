<template>
  <div class="layout" :class="{'layout--revision': revisionContent}">
    <div class="layout__panel flex flex--row" :class="{'flex--end': styles.showSideBar}">
      <div class="layout__panel layout__panel--explorer" v-show="styles.showExplorer" :aria-hidden="!styles.showExplorer" :style="{width: styles.layoutOverflow ? '100%' : constants.explorerWidth + 'px'}">
        <explorer></explorer>
      </div>
      <div class="layout__panel flex flex--column" :style="{width: styles.innerWidth + 'px'}">
        <div class="layout__panel layout__panel--navigation-bar" v-show="styles.showNavigationBar" :style="{height: constants.navigationBarHeight + 'px'}">
          <navigation-bar></navigation-bar>
        </div>
        <div class="layout__panel flex flex--row" :style="{height: styles.innerHeight + 'px'}">
          <div class="layout__panel layout__panel--editor" v-show="styles.showEditor" :style="{width: (styles.editorWidth + styles.editorGutterWidth) + 'px', fontSize: styles.fontSize + 'px'}">
            <div class="gutter" :style="{left: styles.editorGutterLeft + 'px'}">
              <div class="gutter__background" v-if="styles.editorGutterWidth" :style="{width: styles.editorGutterWidth + 'px'}"></div>
            </div>
            <editor></editor>
            <div class="gutter" :style="{left: styles.editorGutterLeft + 'px'}">
              <sticky-comment v-if="styles.editorGutterWidth && stickyComment === 'top'"></sticky-comment>
              <current-discussion v-if="styles.editorGutterWidth"></current-discussion>
            </div>
          </div>
          <div class="layout__panel layout__panel--button-bar" v-show="styles.showEditor" :style="{width: constants.buttonBarWidth + 'px'}">
            <button-bar></button-bar>
          </div>
          <div class="layout__panel layout__panel--preview" v-show="styles.showPreview" :style="{width: (styles.previewWidth + styles.previewGutterWidth) + 'px', fontSize: styles.fontSize + 'px'}">
            <div class="gutter" :style="{left: styles.previewGutterLeft + 'px'}">
              <div class="gutter__background" v-if="styles.previewGutterWidth" :style="{width: styles.previewGutterWidth + 'px'}"></div>
            </div>
            <preview></preview>
            <div class="gutter" :style="{left: styles.previewGutterLeft + 'px'}">
              <sticky-comment v-if="styles.previewGutterWidth && stickyComment === 'top'"></sticky-comment>
              <current-discussion v-if="styles.previewGutterWidth"></current-discussion>
            </div>
          </div>
          <div class="layout__panel layout__panel--find-replace" v-if="showFindReplace">
            <find-replace></find-replace>
          </div>
        </div>
        <div class="layout__panel layout__panel--status-bar" v-show="styles.showStatusBar" :style="{height: constants.statusBarHeight + 'px'}">
          <status-bar></status-bar>
        </div>
      </div>
      <div class="layout__panel layout__panel--side-bar" v-show="styles.showSideBar" :style="{width: styles.layoutOverflow ? '100%' : constants.sideBarWidth + 'px'}">
        <side-bar></side-bar>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapGetters, mapActions } from 'vuex';
import NavigationBar from './NavigationBar';
import ButtonBar from './ButtonBar';
import StatusBar from './StatusBar';
import Explorer from './Explorer';
import SideBar from './SideBar';
import Editor from './Editor';
import Preview from './Preview';
import StickyComment from './gutters/StickyComment';
import CurrentDiscussion from './gutters/CurrentDiscussion';
import FindReplace from './FindReplace';
import editorSvc from '../services/editorSvc';

export default {
  components: {
    NavigationBar,
    ButtonBar,
    StatusBar,
    Explorer,
    SideBar,
    Editor,
    Preview,
    StickyComment,
    CurrentDiscussion,
    FindReplace,
  },
  computed: {
    ...mapState('content', [
      'revisionContent',
    ]),
    ...mapState('discussion', [
      'stickyComment',
    ]),
    ...mapGetters('layout', [
      'constants',
      'styles',
    ]),
    showFindReplace() {
      return !!this.$store.state.findReplace.type;
    },
  },
  methods: {
    ...mapActions('layout', [
      'updateBodySize',
    ]),
    saveSelection: () => editorSvc.saveSelection(true),
  },
  created() {
    this.updateBodySize();
    window.addEventListener('resize', this.updateBodySize);
    window.addEventListener('keyup', this.saveSelection);
    window.addEventListener('mouseup', this.saveSelection);
    window.addEventListener('focusin', this.saveSelection);
    window.addEventListener('contextmenu', this.saveSelection);
  },
  mounted() {
    const editorElt = this.$el.querySelector('.editor__inner');
    const previewElt = this.$el.querySelector('.preview__inner-2');
    const tocElt = this.$el.querySelector('.toc__inner');
    editorSvc.init(editorElt, previewElt, tocElt);

    // Focus on the editor every time reader mode is disabled
    this.$watch(() => this.styles.showEditor,
      showEditor => showEditor && editorSvc.clEditor.focus());
  },
  destroyed() {
    window.removeEventListener('resize', this.updateStyle);
    window.removeEventListener('keyup', this.saveSelection);
    window.removeEventListener('mouseup', this.saveSelection);
    window.removeEventListener('focusin', this.saveSelection);
    window.removeEventListener('contextmenu', this.saveSelection);
  },
};
</script>

<style lang="scss">
@import 'common/variables.scss';

.layout {
  position: absolute;
  width: 100%;
  height: 100%;
}

.layout__panel {
  position: relative;
  width: 100%;
  height: 100%;
  flex: none;
  overflow: hidden;
}

.layout__panel--navigation-bar {
  background-color: $navbar-bg;
}

.layout__panel--status-bar {
  background-color: #007acc;
}

.layout__panel--editor {
  background-color: $editor-background-light;

  .app--dark & {
    background-color: $editor-background-dark;
  }

  .gutter__background,
  .comment-list__current-discussion,
  .sticky-comment,
  .current-discussion {
    background-color: mix(#000, $editor-background-light, 6.7%);
    border-color: $editor-background-light;

    .app--dark & {
      background-color: mix(#fff, $editor-background-dark, 6.7%);
      border-color: $editor-background-dark;
    }
  }
}

$preview-background-light: #f3f3f3;
$preview-background-dark: #252525;

.layout__panel--preview,
.layout__panel--button-bar {
  background-color: $preview-background-light;

  .app--dark & {
    background-color: $preview-background-dark;
  }
}

.layout__panel--preview {
  .gutter__background,
  .comment-list__current-discussion,
  .sticky-comment,
  .current-discussion {
    background-color: mix(#000, $preview-background-light, 6.7%);
    border-color: $preview-background-light;
  }
}

.layout__panel--explorer,
.layout__panel--side-bar {
  background-color: #dadada;
}

.layout__panel--find-replace {
  background-color: #e6e6e6;
  position: absolute;
  left: 0;
  bottom: 0;
  width: 300px;
  height: auto;
  border-top-right-radius: $border-radius-base;
}
</style>