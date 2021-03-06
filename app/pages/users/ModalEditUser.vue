<template>
  <Modal :isOpen="model" @onSubmit="submit" type="lg">
    <div slot="content" class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <div class="list_header">
            <div class="flex">
              <span>Редактировать сотрудника</span>
              <div class="buttons">
                <button class="button-top close" type="button" @click="close"></button>
              </div>
            </div>
          </div>
        </div>
        <div class="profile full modal-body">
          <div class="flex column">
            <div class="form-item">
              <div :class="['form-group', {'has-error': errors.has('login')}]">
                <label for="field-login">Логин *</label>
                <input id="field-login" v-validate="'required'" v-model="model.login" />
                <span v-show="errors.has('login')" class="help-block">{{ errors.first('login') }}</span>
              </div>
              <div :class="['form-group', {'has-error': errors.has('password')}]">
                <label for="field-password">Пароль</label>
                <input id="field-password" name="password" type="password" v-model="model.password">
                <span v-show="errors.has('password')" class="help-block">{{ errors.first('password') }}</span>
              </div>
              <div :class="['form-group', {'has-error': errors.has('fullname')}]">
                <label for="field-fullname">ФИО *</label>
                <input id="field-fullname" v-validate="'required'" name="fullname" v-model="model.fullname">
                <span v-show="errors.has('fullname')" class="help-block">{{ errors.first('fullname') }}</span>
              </div>
              <div :class="['form-group', {'has-error': errors.has('email')}]">
                <label for="field-email">Email *</label>
                <input id="field-email" v-validate="'required|email'" name="email" v-model="model.email">
                <span v-show="errors.has('email')" class="help-block">{{ errors.first('email') }}</span>
              </div>
            </div>
            <div class="form-item">
              <div :class="['form-group', {'has-error': errors.has('dept')}]">
                <label for="field-dept">Департамент</label>
                <el-cascader :options="departments" v-model="model.deptHierarchy" change-on-select @change="setVal"></el-cascader>
                <span v-show="errors.has('dept')" class="help-block">{{ errors.first('dept') }}</span>
              </div>
              <div :class="['form-group input-exc', {'has-error': errors.has('position')}]">
                <label for="field-position">Должность</label>
                <Multiselect
                  id="field-position"
                  name="position"
                  placeholder="Выберите должность"
                  v-model="model.position"
                  :options="filteredPositions"
                  track-by="name"
                  label="name">
                </Multiselect>
                <span v-show="errors.has('position')" class="help-block">{{ errors.first('position') }}</span>
              </div>
              <div :class="['form-group', {'has-error': errors.has('phone')}]">
                <label for="field-phone">Телефон *</label>
                <masked-input id="field-phone" mask="\+1 (111) 111-11-11" name="phone" v-validate="'required'" v-model="model.phone"></masked-input>
                <span v-show="errors.has('phone')" class="help-block">{{ errors.first('phone') }}</span>
              </div>
              <div class="flex flex-end">
                <button type="submit" class="save pad2">Сохранить</button>
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer"></div>
      </div>
    </div>
  </Modal>
</template>

<script>
import Modal from '@/Modal'
import InputBase from '@/Input'
import MaskedInput from 'vue-masked-input'
import Multiselect from 'vue-multiselect'

export default {
  components: {
    Modal,
    MaskedInput,
    InputBase,
    Multiselect
  },
  props: ['isOpen', 'model', 'departments', 'onSubmit', 'onClose', 'otdels'],
  data () {
    return {
      filteredOtdels: [],
      filteredPositions: [],
      positions: []
    }
  },
  methods: {
    setVal (val) {
      this.model.department = val[val.length - 1]
      this.filteredPositions = this.positions.filter(item => item.department ? item.department._id === this.model.department : false)
      this.model.deptHierarchy = val
    },
    close () {
      this.$emit('onClose')
    },
    submit () {
      this.$validator.validateAll().then(() => {
        if (this.$_.find(this.$props.users, u => u.login === this.$props.model.login)) {
          this.errors.items.push({
            field: 'login',
            scope: null,
            msg: 'Пользователь с таким логином уже существует'
          })
        } else if (!this.errors.has('login')) {
          this.errors.items = this.$_.reject(this.errors.items, e => e.field === 'login')
        }
        if (this.$_.find(this.$props.users, u => u.email === this.$props.model.email)) {
          this.errors.items.push({
            field: 'email',
            scope: null,
            msg: 'Пользователь с такой почтой уже существует'
          })
        } else if (!this.errors.has('email')) {
          this.errors.items = this.$_.reject(this.errors.items, e => e.field === 'email')
        }
        if (!this.$_.size(this.errors.items)) {
          this.$emit('onSubmit', this.model)
        }
      }).catch(() => {
      })
    },
    loadPositions () {
      this.$api('get', 'positions').then(response => {
        this.positions = response.data.positions
      })
    }
  },
  mounted () {
    this.loadPositions()
  },
  watch: {
    'model.department': function (val) {
      if (val) {
        this.filteredOtdels = this.otdels.filter(item => item.parent === val._id)
      } else {
        this.filteredOtdels = []
      }
    },
    'model.otdel': function (val) {
      if (val) {
        this.filteredPositions = this.positions.filter(item => item.department._id === val._id || item.department._id === (this.model.department && this.model.department._id))
      } else {
        this.filteredPositions = []
      }
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
