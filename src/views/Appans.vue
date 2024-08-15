<template>
    <section>
      <div class="list-group" id="list-tab" role="tablist">
        <ul class="nav nav-tabs">
          <template v-for="(item, index) in statusList" :key="index">
            <li class="nav-item">
              <a
                class="nav-link"
                :class="{ active: index === select, disabled: item.disabled }"
                data-bs-toggle="list"
                :href="`#list-${item.id}`"
                @click="select = index"
                >{{ item.title }}</a
              >
            </li>
          </template>
        </ul>
      </div>
      <div>
        <div class="tab-content" id="nav-tabContent">
          <div
            class="tab-pane fade p-5 bg-white rounded-bottom-3"
            :class="{ show: select === 0, active: select === 0 }"
            id="list-signIn"
            role="tabpanel"
            aria-labelledby="list-home-list"
          >
            <Signin />
          </div>
          <div
            class="tab-pane fade p-5 bg-white rounded-bottom-3"
            :class="{ show: select === 1, active: select === 1 }"
            id="list-signUp"
            role="tabpanel"
            aria-labelledby="list-profile-list"
          >
            <Signup />
          </div>
          <div
            class="tab-pane fade p-5 bg-white rounded-bottom-3"
            :class="{ show: select === 2, active: select === 2 }"
            id="list-checkOut"
            role="tabpanel"
            aria-labelledby="list-messages-list"
          >
            <Todolist />
          </div>
          <div
            class="tab-pane fade p-5 bg-white rounded-bottom-3"
            :class="{ show: select === 3, active: select === 3 }"
            id="list-signOut"
            role="tabpanel"
            aria-labelledby="list-settings-list"
          >
            <section class="d-flex justify-content-center">
              <div class="card border-0" style="width: 24rem">
                <button class="btn btn-dark" type="button" @click="signOut">
                  <span role="status">Sign Out</span>
                </button>
              </div>
            </section>
          </div>
        </div>
      </div>
    </section>
    <section></section>
  </template>
  
  <style lang="scss">
  .nav-tabs .nav-item.show .nav-link,
  .nav-tabs .nav-link {
    color: #fff;
  }
  .nav-tabs .nav-item.show .nav-link,
  .nav-tabs .nav-link.active {
    color: #000;
  }
  .disabled {
    opacity: 0.5;
  }
  </style>
  
  <script setup>
  import { ref, onMounted, inject } from 'vue'
  import { apiCheckOut, apiPostSignOut } from '../assets/javascript/api'
  import Signin from '../components/week2/SignIn.vue'
  import Signup from '../components/week2/SignUp.vue'
  import Todolist from '../components/week2/TodoList.vue'
  const emitter = inject('emitter')
  const select = ref(0)
  const hakkaCookie = ref(
    document.cookie.replace(
      // eslint-disable-next-line no-useless-escape
      /(?:(?:^|.*;\s*)hexschoolTodoHakka\s*\=\s*([^;]*).*$)|^.*$/,
      '$1'
    )
  )
  const statusList = ref([
    { title: '登入', id: 'signIn', disabled: false },
    { title: '註冊', id: 'signUp', disabled: false },
    { title: '代辦事項', id: 'checkOut', disabled: true },
    { title: '登出', id: 'signOut', disabled: true }
  ])
  const checkOut = () => {
    apiCheckOut({
      headers: {
        Authorization: hakkaCookie.value
      }
    })
      .then((res) => {
        console.log(res)
        select.value = 2
        statusList.value.forEach((item) => {
          item.disabled = !item.disabled
        })
      })
      .catch((err) => {
        console.log(err)
      })
  }
  const signOut = () => {
    apiPostSignOut(
      {},
      {
        headers: {
          Authorization: hakkaCookie.value
        }
      }
    )
      .then((res) => {
        console.log(res)
        emitter.emit('messageModal', {
          status: true,
          message: '登出成功'
        })
        select.value = 0
        statusList.value.forEach((item) => {
          item.disabled = !item.disabled
        })
      })
      .catch((err) => {
        emitter.emit('messageModal', {
          status: false,
          message: err.response.data.message
        })
      })
  }
  onMounted(() => {
    emitter.on('checkOutMitt', () => {
      hakkaCookie.value = document.cookie.replace(
        // eslint-disable-next-line no-useless-escape
        /(?:(?:^|.*;\s*)hexschoolTodoHakka\s*\=\s*([^;]*).*$)|^.*$/,
        '$1'
      )
      checkOut()
    })
    checkOut()
  })
  </script>