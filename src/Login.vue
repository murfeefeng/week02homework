<script setup>
import axios from 'axios'
import { ref } from 'vue'

const apiURL = 'https://todolist-api.hexschool.io'
const signInToken = ref(null)

//註冊
const signUpField = ref({
  email: '',
  password: '',
  nickname: ''
})
const signUpUID = ref('')

// console.log(signUpField)
const signUpPost = async () => {
  // console.log(res)
  try {
    const res = await axios.post(`${apiURL}/users/sign_up`, signUpField.value)
    console.log(res)
    signUpUID.value = res.data.uid
  } catch (error) {
    console.log(error.message)
    // errMsg.value = res.message
  }
}

//登入----------------------

const signInField = ref({
  email: '',
  password: ''
})

const signStatus = ref()
const signInOK = ref('')
const errMsg = ref('')
const signInPost = async () => {
  try {
    const res = await axios.post(`${apiURL}/users/sign_in`, signInField.value)
    // console.log(res)
    signStatus.value = res.data.status
    signInToken.value = res.data.token
    signInOK.value = `Hello! ` + res.data.nickname + `! 登入成功囉！`
    alert(signInOK.value)
    // console.log('hi', signUpField.value)

    document.cookie = `customToken=${res.data.token}; expires=${res.data.exp}; path=/`
    // console.log('UID:', checkUser.value.uid)
    signCheck()
  } catch (error) {
    console.log(error)
    errMsg.value = error.response.data.message
  }
}

//驗證----------------------

const checkUser = ref({
  nickname: '',
  uid: ''
})

const signCheck = async () => {
  signInToken.value = document.cookie.replace(
    /(?:(?:^|.*;\s*)customToken\s*\=\s*([^;]*).*$)|^.*$/,
    '$1'
  )

  if (!signInToken.value) {
    console.log('尚未登入！')
    return //沒有 token
  }
  const res = await axios.get(`${apiURL}/users/checkout`, {
    headers: {
      Authorization: signInToken.value
    }
  })
  checkUser.value = res.data

  getTodosData()
}

signCheck()

// console.log(checkUser.value.uid)
//登出-------
const signOutCheck = ref('')
const signoutPost = async () => {
  // console.log(signInToken.value)
  try {
    const res = await axios.post(
      `${apiURL}/users/sign_out`,
      {},
      {
        headers: {
          Authorization: signInToken.value
        }
      }
    )
    document.cookie = 'customToken=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/'
    signOutCheck.value = '登出成功！'
    alert(signOutCheck.value)
    checkUser.value.uid = ''
    console.log(res)
  } catch (error) {
    console.log(error)
  }
}

//抓原本的todos
const getTodo = ref('')
const todoMsg = ref('')
const getTodosData = async () => {
  try {
    const res = await axios.get(`${apiURL}/todos/`, {
      headers: {
        Authorization: signInToken.value
      }
    })
    // console.log(res.data.data)
    if (res.data.status) {
      if (res.data.data.length === 0) {
        todoMsg.value = '無待辦事項'
      } else {
        todoMsg.value = ''
        // console.log(res.data.data)
        res.data.data.forEach((item) => {
          const createTime = item.createTime
          const date = new Date(createTime * 1000)
          const formattedDate = date.toISOString().slice(0, 19).replace('T', ' ')
          item.createTime = formattedDate
        })

        getTodo.value = res.data.data
      }
    } else {
      todoMsg.value = '無待辦事項'
    }
  } catch (error) {
    console.log(error)
    errMsg.value = error.response?.data?.message || '抓清單失敗'
  }
}

getTodosData()
//新增
const todoField = ref('')
const newTodo = ref({
  content: ''
})
const addTodos = async () => {
  try {
    newTodo.value.content = todoField.value
    const res = await axios.post(`${apiURL}/todos/`, newTodo.value, {
      headers: {
        Authorization: signInToken.value
      }
    })
    console.log('新增成功', res)
    getTodosData()
  } catch (error) {
    console.log(error)
    errMsg.value = error.response?.data?.message || '新增失敗'
  }
}

//確認清單項目
// const newTodoStatus = ref(false)
const toggleTodos = async (id, status) => {
  try {
    const res = await axios.patch(
      `${apiURL}/todos/${id}/toggle`,
      {
        status: status
      },
      {
        headers: {
          Authorization: signInToken.value
        }
      }
    )
    console.log('修改確認', res)
    getTodosData()
  } catch (error) {
    console.log(error)
    errMsg.value = error.response?.data?.message || '修改確認失敗'
  }
}

//刪除
const deleteTodos = async (id) => {
  try {
    const res = await axios.delete(`${apiURL}/todos/${id}`, {
      headers: {
        Authorization: signInToken.value
      }
    })

    console.log('刪除成功', res)
    getTodosData()
  } catch (error) {
    console.log(error)
    errMsg.value = error.response?.data?.message || '刪除失敗'
  }
}

//修改項目
// const openThis = ref(false)
const editField = ref('')
const editIndex = ref(null)
const openField = (id, index) => {
  // openThis.value = true
  // console.log(id)
  // console.log(index)
  editIndex.value = index
  getTodo.value.forEach((item) => {
    if (item.id === id) {
      if (!editField.value == '') {
        item.content = editField
        console.log(item)
      }
    }
  })
}

const cancelField = () => {
  editIndex.value = null
}
const saveField = async (id, index) => {
  try {
    console.log(editField.value)

    getTodo.value[index].content = editField.value
    editIndex.value = null
    const res = await axios.put(
      `${apiURL}/todos/${id}`,
      {
        content: getTodo.value[index].content
      },
      {
        headers: {
          Authorization: signInToken.value
        }
      }
    )
    console.log('修改清單完成', res)
    getTodosData()
  } catch (error) {
    console.log(error)
    errMsg.value = error.response?.data?.message || '修改清單失敗'
  }
}
</script>

<template>
  <div class="container">
    <!-- Check Out -->
    <section>
      <h3 class="mb-4">驗證</h3>
      <div class="mb-3">
        <div v-if="checkUser.uid">
          <!-- <p>{{ signInOK }}</p> -->
          <h4>Hello!{{ checkUser.nickname }}!</h4>
          <!-- <h4>UID:{{ checkUser.uid }}</h4> -->
          <button type="button" class="btn btn-primary" @click="signoutPost">Sign Out</button>
        </div>
        <div v-else>您還沒有登入喔～</div>
        <!-- {{ signOutCheck }} -->
      </div>
    </section>
    <hr />

    <!-- Sign In -->
    <section>
      <h3 class="mb-4">登入</h3>
      <div class="row">
        <div class="col-3">
          <div class="mb-3">
            <input
              type="email"
              class="form-control"
              placeholder="Email"
              v-model="signInField.email"
            />
          </div>
        </div>
        <div class="col-3">
          <div class="mb-3">
            <input
              type="password"
              class="form-control"
              placeholder="Password"
              v-model="signInField.password"
            />
          </div>
        </div>
        <div class="col-3">
          <button type="button" class="btn btn-primary" @click="signInPost">Sign In</button>
        </div>
      </div>

      <!-- {{ signInField }} -->
      <div v-if="signInOK">
        <!-- <p>{{ signInToken }}</p> -->
      </div>
      <div v-else>
        <small class="text-danger">{{ errMsg }}</small>
      </div>
    </section>
    <hr />

    <!-- Sign Up -->
    <section>
      <h3 class="mb-4">註冊</h3>
      <div class="row">
        <div class="col-3">
          <div class="mb-3">
            <input
              type="email"
              class="form-control"
              placeholder="Email"
              v-model="signUpField.email"
            />
          </div>
        </div>
        <div class="col-3">
          <div class="mb-3">
            <input
              type="password"
              class="form-control"
              placeholder="Password"
              v-model="signUpField.password"
            />
          </div>
        </div>
        <div class="col-3">
          <div class="mb-3">
            <input
              type="text"
              class="form-control"
              placeholder="Nick Name"
              v-model="signUpField.nickname"
            />
          </div>
        </div>
        <div class="col-3">
          <button type="button" class="btn btn-primary" @click="signUpPost">Sign Up</button>
        </div>
      </div>
      <!-- {{ signUpField }} -->
      <div v-if="signUpUID">
        <small>UID:{{ signUpUID }}</small>
      </div>
    </section>
    <hr />

    <!-- Todo List-->
    <section v-if="checkUser.uid">
      <h1>TODO LIST</h1>

      <input type="text" placeholder="ＴＯＤＯ清單內容" v-model="todoField" />
      {{ todoField }}
      <button type="button" @click="addTodos(todoField)">新增</button>
      <hr />
      <p>{{ todoMsg }}</p>

      {{ getTodo }}
      <ul class="todoList">
        <li v-for="(list, index) in getTodo" :key="list.id">
          <div>
            <input
              class="form-check-input"
              type="checkbox"
              value=""
              :id="list.id"
              v-model="list.status"
              @change="toggleTodos(list.id)"
            />
            <label class="form-check-label" :for="list.id" v-if="index !== editIndex">
              {{ list.content }}
            </label>

            <div class="editBox" v-else>
              <input type="text" v-model="editField" />
              {{ editField }}
              <button
                type="button"
                class="btn btn-outline-secondary btn-sm mx-2"
                @click="cancelField(index)"
              >
                取消
              </button>
              <button
                type="button"
                class="btn btn-outline-primary btn-sm mx-2"
                @click="saveField(list.id, index)"
              >
                儲存
              </button>
            </div>

            <button
              type="button"
              class="btn btn-primary mx-2"
              @click="openField(list.id, index)"
              v-if="index !== editIndex"
            >
              修改
            </button>
            <button
              type="button"
              class="btn btn-danger"
              @click="deleteTodos(list.id)"
              v-if="index !== editIndex"
            >
              刪除
            </button>
          </div>
          <small class="text-black-50">{{ list.createTime }}</small>
        </li>
      </ul>
    </section>
  </div>
</template>
