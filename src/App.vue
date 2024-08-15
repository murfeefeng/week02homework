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
    console.log(res)
    signStatus.value = res.data.status
    signInToken.value = res.data.token
    signInOK.value = 'Hello' + signUpField.value.nickname + '!'
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
    return // 如果沒有 token，可以選擇直接返回或執行其他邏輯
  }
  const res = await axios.get(`${apiURL}/users/checkout`, {
    headers: {
      Authorization: signInToken.value
    }
  })
  checkUser.value = res.data
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
    checkUser.value.uid = ''
    console.log(res)
  } catch (error) {
    console.log(error)
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
        <p>{{ signInOK }}</p>
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

    <h1>TODO LIST</h1>
  </div>
</template>
