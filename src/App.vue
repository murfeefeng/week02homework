<template>
  <div class="container">
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
      {{ signUpField }}
      <h4>UID:{{ signUpUID }}</h4>
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
      {{ signInField }}
      <h4>TOKEN:{{ signInToken }}</h4>
      <h4>EXP:{{ signInEXP }}</h4>
    </section>
    <hr />

    <!-- Check Out -->
    <section>
      <h3 class="mb-4">驗證</h3>
      <div class="mb-3">
        <div v-if="checkUser.uid">
          <h4>Hello!{{ checkUser.nickname }}!</h4>
          <h4>UID:{{ checkUser.uid }}</h4>
        </div>
        <div v-else>您還沒有登入喔～</div>
      </div>
    </section>
    <hr />
    <!-- Check Out -->
    <section>
      <h3 class="mb-4">登出</h3>
      <div class="row">
        <div class="col-3">
          <div class="mb-3">
            <input
              type="text"
              class="form-control"
              id="formGroupExampleInput"
              placeholder="Token"
            />
          </div>
        </div>
        <div class="col-3">
          <button type="button" class="btn btn-primary">Sign Out</button>
        </div>
      </div>
    </section>

    <h1>TODO LIST</h1>
  </div>
</template>

<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'

const apiURL = 'https://todolist-api.hexschool.io'

//驗證
const checkUser = ref({
  nickname: '',
  uid: ''
})

onMounted(async () => {
  const myCookie = document.cookie.replace(
    /(?:(?:^|.*;\s*)customToken\s*\=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  // console.log(myCookie)

  const res = await axios.get(`${apiURL}/users/checkout`, {
    headers: {
      Authorization: myCookie
    }
  })
  checkUser.value = res.data
  // console.log(checkUser.value)
})

//登入

const signInField = ref({
  email: '',
  password: ''
})

const signInToken = ref('')
const signInEXP = ref('')
const signInPost = async () => {
  try {
    const res = await axios.post(`${apiURL}/users/sign_in`, signInField.value)
    console.log(res)
    signInToken.value = res.data.token
    signInEXP.value = res.data.exp
    document.cookie = `customToken=${res.data.token}; expires=${res.data.exp}; path=/`
  } catch (error) {
    console.log(error)
  }
}

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
    console.log(error)
  }
}
</script>
