<template>
  <div class="working_area">
    <div class="white-block no-padding">
      <div class="padding-block">
        <div class="flex margin-bottom align-center">
          <div class="search">
          </div>
          <div class="add" v-if="$auth().hasRole('admin')">
            <button class="add-button" @click="toggleModal('createUser', {})"><img src="~assets/img/add.png">Добавить сотрудника</button>
          </div>
        </div>
        <div class="mob-none">
          <v-client-table ref="table" v-bind="tableData" :data="filteredUsers" :columnsDropdown="true">
            <div class="flex align-center sm-w" slot="tools" slot-scope="props">
              <a @click="toggleModal('showUser', props.row)" class="green_anchor">Подробнее</a>
            </div>
            <div class="border-none" slot="admin" slot-scope="props" v-if="$auth().hasRole('admin')">
              <button class="button-table edit" @click="toggleModal('editUser', $_.clone(props.row))"></button>
              <button class="button-table remove" @click="toggleModal('deleteUser', props.row) "></button>
            </div>
          </v-client-table>
        </div>
        <div class="mob-block">
          <div class="search">
            <input type="text" placeholder="Поиск" name="search" v-model="mobTableData.filter">
          </div>
          <b-table
            stacked
            :current-page="mobTableData.currentPage"
            :filter="mobTableData.filter"
            @filtered="onFiltered"
            :per-page="mobTableData.perPage"
            :items="filteredUsers"
            :fields="mobTableData.fields">
            <template slot="fullname" slot-scope="row">
              <span>{{row.value}} (<a @click="toggleModal('showUser', row.item)" class="green_anchor">Подробнее</a>)</span>
            </template>
            <template slot="actions" slot-scope="row" v-if="$auth().hasRole('admin')">
              <button class="button-table edit" @click="toggleModal('editUser', $_.clone(row.item))"></button>
              <button class="button-table remove" @click="toggleModal('deleteUser', row.item) "></button>
            </template>
          </b-table>
          <b-pagination :total-rows="mobTableData.totalRows" :per-page="mobTableData.perPage" v-model="mobTableData.currentPage"/>
        </div>
      </div>
    </div>
    <ModalCreateUser v-if="modal.createUser" :model="modal.createUser" :users="users" :departments="group(departments)" :otdels="otdels" :positions="positions" @onSubmit="createUser" @onClose="toggleModal('createUser')" :btnDisabled="btnDisabled"></ModalCreateUser>
    <ModalDeleteUser :model="modal.deleteUser" @onSubmit="deleteUser" @onClose="toggleModal('deleteUser')"></ModalDeleteUser>
    <ModalEditUser v-if="modal.editUser" :model="modal.editUser" :departments="group(departments)" :otdels="otdels" :positions="positions" @onSubmit="editUser" @onClose="toggleModal('editUser')"></ModalEditUser>
    <ModalShowUser v-if="modal.showUser" :model="modal.showUser" @onClose="toggleModal('showUser')"></ModalShowUser>
    <ModalCreateTask :model="modal.createTask" :users="users" @onSubmit="createTask" @onClose="toggleModal('createTask')"></ModalCreateTask>
  </div>
</template>

<script>
import ModalCreateUser from './users/ModalCreateUser'
import ModalEditUser from './users/ModalEditUser'
import ModalShowUser from './users/ModalShowUser'
import ModalDeleteUser from './users/ModalDeleteUser'
import ModalCreateTask from './tasks/ModalCreateTask'
import bTable from 'bootstrap-vue/es/components/table/table'
import bPagination from 'bootstrap-vue/es/components/pagination/pagination'

export default {
  plugins: ['auth'],
  components: {
    ModalCreateUser,
    ModalDeleteUser,
    ModalEditUser,
    ModalCreateTask,
    ModalShowUser,
    'b-table': bTable,
    'b-pagination': bPagination
  },
  data () {
    return {
      seoTitle: this.$trans('pages.index.seoTitle'),
      users: [],
      allDepartments: [],
      departments: [],
      storeDepartments: this.$store.getters['app/departments'],
      otdels: [],
      positions: [],
      filter: false,
      modal: {
        editUser: false,
        showUser: false,
        createTask: false,
        createUser: false,
        deleteUser: false
      },
      btnDisabled: false,
      tableData: {
        columns: ['id', 'fullname', 'posName', 'deptName', 'phone', 'email', 'tools', 'admin'],
        options: {
          headings: {
            id: 'ID',
            fullname: 'ФИО',
            posName: 'Должность',
            deptName: 'Отдел',
            phone: 'Телефон',
            email: 'Email',
            tools: 'Доп. информация',
            admin: 'Управление'
          },
          orderBy: {
            column: 'id',
            ascending: false
          },
          sortable: ['id', 'fullname', 'posName', 'deptName', 'phone', 'email'],
          filterable: ['id', 'fullname', 'posName', 'deptName', 'phone', 'email'],
          customSorting: {
            id: function (ascending) {
              return (a, b) => {
                a = a.id * 1
                b = b.id * 1
                if (ascending) return a >= b ? 1 : -1
                return a <= b ? 1 : -1
              }
            }
          },
          columnsClasses: {
            admin: 'admin',
            phone: 'w130',
          },
          rowClasses: {
            admin: 'admin',
            phone: 'w130',
          },
          highlightMatches: true
        }
      },
      mobTableData: {
        fields: [
          { key: 'fullname', label: 'ФИО' },
          { key: 'posName', label: 'Должность' },
          { key: 'deptName', label: 'Департамент' },
          { key: 'phone', label: 'Телефон'},
          { key: 'email', label: 'E-mail' },
          { key: 'actions', label: 'Действия' }
        ],
        currentPage: 1,
        perPage: 5,
        totalRows: 0,
        filter: null
      }
    }
  },
  computed: {
    filteredUsers: {
      get: function () {
        let users = _.clone(this.users)
        if (this.filter !== false) {
          users = _.filter(users, ['department', this.filter])
        }

        const departments = {}
        if (this.allDepartments && this.allDepartments.length > 0) {
          _.clone(this.allDepartments).forEach(item => {
            departments[item._id] = item.name
          })
        }
        const positions = {}
        if (this.positions && this.positions.length > 0) {
          _.clone(this.positions).forEach(item => {
            positions[item._id] = item.name
          })
        }

        users = users.map(item => {
          item.deptName = departments[item.department]
          item.posName = positions[item.position]
          return item
        })
        return users
      }
    }
  },
  methods: {
    toggleModal (name, model) {
      this.modal[name] = model === undefined ? !this.modal[name] : model
    },
    async createUser (user) {
      try {
        const response = await this.$api('post', 'users', user)
        this.loadUsers()
        this.modal.createUser = false
        this.notify(response.data.message)
      } catch (e) {
        this.notify('Временно нельзя создать пользователя', 'info')
        this.$log(e, 'danger')
      } 
      finally {
        this.btnDisabled = false
      }
      
    },
    editUser (user) {
      this.$api('put', 'users/' + user._id, user).then(response => {
        this.loadUsers()
        this.modal.editUser = false
        this.notify(response.data.message)
      }).catch(e => {
        this.notify('Временно нельзя редактировать пользователя', 'info')
        this.modal.editUser = false
        this.$log(e, 'danger')
      })
    },
    deleteUser (user) {
      this.$api('delete', 'users/' + user._id).then(response => {
        this.users = this.$_.remove(this.users, u => u._id !== user._id)
        this.modal.deleteUser = false
        this.notify(response.data.message)
      }).catch(e => {
        this.notify('Временно нельзя удалить пользователя', 'info')
        this.modal.deleteUser = false
        this.$log(e, 'danger')
      })
    },
    createTask (task) {
      task.files = this.$_.map(task.files, f => f.file)
      let data = this.$createFormData(task)
      this.$api('post', 'tasks', data).then(response => {
        this.modal.createTask = false
        this.notify(response.data.message)
      }).catch(e => {
        this.notify('Временно нельзя создать задачу', 'info')
        this.$log(e, 'danger')
      })
    },
    loadUsers () {
      this.$api('get', 'users').then(response => {
        if (response.data && response.data.length > 0) {
          this.users = response.data.filter(user => user._id !== this.$auth().user._id && user.login !== 'admin')
        }
      }).catch(e => {
        if (e.response) {
          return this.notify(e.response.data.message, 'danger')
        }
        this.notify('Временная ошибка', 'danger')
      })
    },
    loadDepartments () {
      this.$api('get', 'departments').then(response => {
        this.allDepartments = response.data.departments
        this.departments = response.data.departments.filter(
          item => item.departmentType === 'head'
        )
        this.otdels = response.data.departments.filter(item => item.departmentType === 'common')
        let sidebar = [
          {
            link: { name: 'users' },
            isActive: () => this.$isRoute('users'),
            name: 'Все',
            imgSrc: 'left_menu/1.png'
          }
        ]
        sidebar = [...sidebar, ...this.group(this.departments)]
        this.$store.commit('app/setSidebar', sidebar)
      }).catch(e => {
        if (e.response) {
          return this.notify(e.response.data.message, 'danger')
        }
        this.notify('Временная ошибка', 'danger')
      })
    },
    group (array) {
      let heads = array.map(item => {
        item.value = item._id
        item.label = item.name
        item.link = { name: 'usersByDep', params: { param1: item._id }}
        item.isActive = () => {
          const children = item.children && item.children.map(c => c._id) || []
          return this.$isRoute('usersByDep', 'param1', item._id) || (children.length && children.some(elem => this.$isRoute('usersByDep', 'param1', elem)))
        }
        item.imgSrc = 'left_menu/1.png'
        item.imgSrc2 = 'folder-h.png'
        return item
      })
      let result = []
      this.otdels.forEach(item => {
        this.setChildren(heads, { ...item })
      })
      result = [...heads]
      return result
    },
    exists (arr, key, val) {
      return arr.filter(item => item[key] === val).length > 0
    },
    setChildren (arr, item) {
      const target = JSON.parse(JSON.stringify(item))
      for (let i = 0; i < arr.length; i++) {
        if (arr[i].children) {
          if (arr[i]._id === target.parent) {
            if (!this.exists(arr[i].children, '_id', target._id)) {
              target.label = target.name
              target.value = target._id
              target.link = { name: 'usersByDep', params: { param1: target._id }}
              target.isActive = () => {
                return this.$isRoute('usersByDep', 'param1', item._id)
              }
              target.imgSrc = 'folder.png'
              target.imgSrc2 = 'folder-h.png'
              arr[i].children.push(target)
            }
          } else {
            this.setChildren(arr[i].children, target)
          }
        } else {
          if (arr[i]._id === target.parent) {
            target.label = target.name
            target.value = target._id
            target.link = { name: 'usersByDep', params: { param1: target._id }}
            target.isActive = () => this.$isRoute('usersByDep', 'param1', item._id)
            target.imgSrc2 = 'folder-h.png'
            target.imgSrc = 'folder.png'

            arr[i].children = [target]
          }
        }
      }
    },
    loadPositions () {
      this.$api('get', 'positions?all=true').then(response => {
        this.positions = response.data.positions
      })
    },
    departmentName (_id) {
      if (this.allDepartments.length > 0) {
        if (this.allDepartments.filter(item => item._id === _id).length > 0) {
          return this.allDepartments.filter(item => item._id === _id)[0].name
        }
      }
    },
    positionName (_id) {
      if (this.positions.length > 0) {
        if (this.positions.filter(item => item._id === _id).length > 0) {
          return this.positions.filter(item => item._id === _id)[0].name
        }
      }
    },
    updateFilter () {
      let department = this.$route.params.param1
      if (department === undefined) {
        department = false
      }
      this.filter = department
    },
    onFiltered (filteredItems) {
      this.mobTableData.totalRows = filteredItems.length
      this.mobTableData.currentPage = 1
    }
  },
  mounted () {
    this.loadUsers()
    this.loadDepartments()
    this.loadPositions()
    this.updateFilter()
    this.mobTableData.totalRows = this.filteredUsers.length
  },
  destroyed () {
    this.$store.commit('app/setSidebar', {})
  },
  watch: {
    $route (to, from) {
      this.updateFilter()
    }
  }
}
</script>

<style lang="scss">
body .wrapper .form-control { border-width: 0 0 1px; border-radius: 0; padding-left: 5px; }
.w130 {
  width: 130px !important;
}
</style>
