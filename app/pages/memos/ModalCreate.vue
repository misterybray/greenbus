<template>
  <Modal :isOpen="model" type="lg" @onSubmit="submit">

    <h3 slot="header" class="modal-title">Создать служебную записку</h3>

    <div slot="content" class="row">
      <div class="col-lg-6">
        <div :class="['form-group', {'has-error': errors.has('name')}]">
          <label for="field-name">Тема *</label>
          <input id="field-name" class="form-control" v-validate="'required'" name="name" v-model="model.name">
          <span v-show="errors.has('name')" class="help-block">{{ errors.first('name') }}</span>
        </div>
        <div :class="['form-group', {'has-error': errors.has('text')}]">
          <label for="field-text">Описание *</label>
          <textarea id="field-text" class="form-control" rows="4" v-validate="'required'" name="text" v-model="model.text"></textarea>
          <span v-show="errors.has('text')" class="help-block">{{ errors.first('text') }}</span>
        </div>
      </div>
      <div class="col-lg-6">
        <div :class="['form-group', {'has-error': errors.has('memoTo')}]">
          <label for="field-memoTo">Кому *</label><br />
          <Multiselect
            id="field-memoTo"
            v-model="selectedUsers"
            :options="usersForSelect"
            :close-on-select="false"
            :hide-selected="true"
            :clear-on-select="false"
            :multiple="true"
            track-by="name"
            label="name"
          >
          </Multiselect>
          <span v-show="errors.has('memoTo')" class="help-block">{{ errors.first('memoTo') }}</span>
        </div>
        <div class="form-group">
          <label class="custom-file-label" for="field-files">Прикрепить файлы</label>
          <input type="file" multiple id="field-files" lang="ru" @change="addFiles">
        </div>
      </div>
    </div>

    <div slot="footer">
      <button type="button" class="btn btn-default" data-dismiss="modal" @click="close"><i class="fa fa-times"></i>&nbsp;&nbsp;Отмена</button>
      <button type="submit" class="btn btn-success"><i class="fa fa-check"></i>&nbsp;&nbsp;Создать</button>
    </div>

  </Modal>
</template>

<script>
  import 'element-ui/lib/theme-chalk/index.css'
  import Modal from '@/Modal'
  import MaskedInput from 'vue-masked-input'
  import Datepicker from 'vuejs-datepicker'
  import { Switch } from 'element-ui'
  import Multiselect from 'vue-multiselect'

  export default {
    components: {
      Modal,
      MaskedInput,
      Datepicker,
      'el-switch': Switch,
      Multiselect
    },
    data () {
      return {
        to: null,
      }
    },
    props: ['model', 'users', 'onSubmit', 'onClose',],
    methods: {
      close () {
        this.$emit('onClose')
      },
      submit () {
        this.$validator.validateAll().then(() => {
          if (!this.$_.size(this.errors.items)) {
            let model = this.$_.clone(this.$props.model)
            model.to = this.$_.map(model.to, u => u.user)
            this.$emit('onSubmit', model)
          }
        }).catch(() => {
        })
      },
      getUser (_id) {
        let user = this.$_.find(this.$props.users, u => u._id === _id)
        return user ? user : {}
      },
      addFiles (e) {
        let files = e.target.files || e.dataTransfer.files
        if (!files.length) return

        this.$props.model.files = files
      }
    },
    computed: {
      usersForSelect () {
        return this.$_.map(this.$props.users, u => {
          return {name: u.fullname, _id: u._id}
        })
      },
      selectedUsers: {
        get: function () {
          if (this.$_.size(this.$props.model.to) > 10) {
            this.errors.items.push({
              field: 'memoTo',
              scope: null,
              msg: 'Допустимо не больше 10 согласующих',
            })
          }
          return this.$_.map(this.$props.model.to, m => {
            return m ? {name: this.getUser(m.user).fullname, _id: m.user} : {}
          })
        },
        set: function (newValue) {
          this.errors.items = this.$_.reject(this.errors.items, e => e.field === 'memoTo')
          this.$props.model.to = this.$_.map(newValue, m => {
            return {user: m._id}
          })
        }
      }
    }
  }
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style lang="scss" scoped>

</style>