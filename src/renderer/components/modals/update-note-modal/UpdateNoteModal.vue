<template src="./UpdateNoteModal.html">

</template>

<script>
  import Vuex from 'vuex';
  import editor from '../../editor/Editor';
  import languages from '../../../assets/data/languages.json';
  import converter from '../../../converter';

  export default {
    name: 'cn-update-note-modal',
    components: {
      editor,
    },
    props: {
      note: Object,
    },
    data() {
      return {
        noteUpdated: {
          name: '',
          description: '',
          files: {},
          updatedAt: null,
          createdAt: null,
          tags: []
        },
        files: [],
        gistFiles: [],
        languages,
        displayDupError: false,
      };
    },
    mounted() {
      if (this.gistsSelected) {
        this.$refs.noteDescription.focus();
      } else {
        this.$refs.noteName.focus();
        this.note.tags = this.note.tags ? this.note.tags : [];
      }

      this.noteUpdated = {...this.note};
      this.noteUpdated.files = {};
      Object.keys(this.note.files).forEach((key, index) => {
        this.files.push({...this.note.files[key], id: index});
        this.gistFiles.push({...this.note.files[key], id: index, deleted: false});
      });
    },
    methods: {
      updateNote() {
        if (!this.containsDupFiles()) {
          if (this.gistsSelected) {
            this.files.forEach(file => {
              if (file.added) {
                this.gistFiles.push(file);
              }
            });

            this.gistFiles.forEach((file, index) => {
              const key = `${this.gistFiles[index].name}.${converter.languageToExtension(this.gistFiles[index].language)}`;

              if (file.deleted) {
                this.noteUpdated.files[key] = null
              } else {
                this.noteUpdated.files[key] = this.files[index];

                const newFile = this.files.filter(f => f.id === file.id)[0];

                if (newFile.name !== file.name || newFile.language !== file.language ) {
                  this.noteUpdated.files[key].filename = `${newFile.name}.${converter.languageToExtension(newFile.language)}`;
                }
              }
            });
          } else {
            this.files.forEach(file => {
              this.noteUpdated.files[file.name] = file;
            });
            this.noteUpdated.updatedAt = new Date();
          }

          this.$store.dispatch('updateNote', this.noteUpdated).then(() => {
            this.$parent.close();
          });
        } else {
          this.displayDupError = true;
        }
      },
      addFile() {
        this.files.push({
          name: '',
          language: 'text',
          content: '',
          deleted: false,
          added: true
        });
      },
      deleteFile(file) {
        if (this.gistsSelected) {
          this.gistFiles.forEach((f, index) => {
            if (f.id === file.id) {
              this.gistFiles[index].deleted = true;
            }
          });
        }
        this.files = this.files.filter(f => f !== file);
      },
      containsDupFiles() {
        const map = new Map();
        let dupFiles = false;

        this.files.forEach(file => {
          const key = `${file.name}.${converter.languageToExtension(
            file.language
          )}`;

          if (map.has(key)) {
            dupFiles = true;
          }
          map.set(key, 1);
        });

        return dupFiles;
      },
    },
    computed: {
      ...Vuex.mapGetters(['gistsSelected']),
      isDisabled() {
        if (this.gistsSelected) {
          return (
            this.files.some(
              file =>
                !/^[^.]*$/.test(file.name) ||
                !/\S/.test(file.name) ||
                !/\S/.test(file.language) ||
                !/\S/.test(file.content)
            )
          )
        }
        return (
          !/\S/.test(this.note.name) ||
          this.files.some(
            file =>
              !/^[^.]*$/.test(file.name) ||
              !/\S/.test(file.name) ||
              !/\S/.test(file.language) ||
              !/\S/.test(file.content)
          )
        );
      },
    },
  };
</script>

<style src="./UpdateNoteModal.scss" lang="scss">

</style>
