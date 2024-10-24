<template>
  <div class="container">
    <div class="g-bg">
      <div class="g-polygon g-polygon-1"></div>
      <div class="g-polygon g-polygon-2"></div>
      <div class="g-polygon g-polygon-3"></div>
    </div>
    <div class="context">
      <div class="login">
        <el-card style="width: 400px">
          <div class="title title-top">登录到</div>
          <div class="title">Start Point</div>

          <el-form ref="formRef" :model="formData" :rules="rules" @keyup.enter="submitForm(formRef)">
            <el-form-item prop="account">
              <el-input v-model="formData.account" placeholder="请输入账户名">
                <template #prefix>
                  <MSIcon name="Monitor" size="16"></MSIcon>
                </template>
              </el-input>
            </el-form-item>

            <el-form-item prop="password">
              <el-input v-model="formData.password" type="password" placeholder="请输入密码">
                <template #prefix>
                  <MSIcon name="Lock" size="16"></MSIcon>
                </template>
              </el-input>
            </el-form-item>

            <el-form-item prop="captcha">
              <el-input v-model="formData.captcha" placeholder="请输入验证码" style="width: 190px;">
                <template #prefix>
                  <MSIcon name="Key" size="16"></MSIcon>
                </template>
              </el-input>
              <!-- 验证码 -->
              <img class="img-code" :src="imgCode" @click="getValidateCode" alt="加载失败。" />
            </el-form-item>

            <el-form-item prop="">
              <el-checkbox v-model="rbPassword"> 记住密码 </el-checkbox>
            </el-form-item>

            <el-form-item>
              <el-button type="primary" @click="submitForm(formRef)">
                登录
              </el-button>
            </el-form-item>
          </el-form>

        </el-card>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, reactive, ref } from 'vue'
import { ElMessage, type FormInstance, type FormRules } from 'element-plus'
import MSIcon from '@/components/MSIcon/index.vue';
import type { LoginParams } from '@/types/auth';
import { getValidateCodeApi } from '@/api/auth';
import { useUserStore } from '@/stores'
import { md5 } from 'js-md5';
import { useRoute, useRouter } from 'vue-router';

const formRef = ref<FormInstance>()
const formData = reactive<LoginParams>({
  account: '',
  password: '',
  captcha: '',
  codeKey: ''
})
const imgCode = ref<string>('')
const rbPassword = ref<boolean>(false)

const rules = reactive<FormRules<LoginParams>>({
  account: [{ required: true, message: '请输入账号', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
  captcha: [{ required: true, message: '请输入验证码', trigger: 'blur' }]
})

const getValidateCode = () => {
  getValidateCodeApi().then((res) => {
    formData.codeKey = res.data.codeKey
    imgCode.value = res.data.codeValue
    formData.captcha = ''
  })
}

const userStore = useUserStore()
const router = useRouter()
const route = useRoute()
const submitForm = async (formEl: FormInstance | undefined) => {
  if (!formEl) return
  await formEl.validate(async (valid, fields) => {
    if (valid) {
      let params = {
        ...formData
      }
      let loginInfo = userStore.loginInfo
      if (loginInfo && loginInfo.password !== params.password) {
        params.password = md5(params.password).toLocaleUpperCase()
      } else if (!loginInfo) {
        params.password = md5(params.password).toLocaleUpperCase()
      }

      await userStore.login(params).then(() => {
        ElMessage({
          message: '登录成功',
          type: 'success',
          plain: true,
        })
        if (rbPassword.value) {
          userStore.setLoginInfo({
            account: params.account,
            password: params.password
          })
        } else {
          userStore.setLoginInfo(null)
        }
        const redirect = route.query.redirect as string
        if (redirect) {
          const redirectUrl = decodeURIComponent(redirect)
          router.push(redirectUrl)
        } else {
          router.push('/')
        }
      }).catch(() => getValidateCode())
    } else {
      ElMessage({
        message: Object.values(fields!)[0][0].message,
        type: 'warning',
        plain: true,
      })
    }
  })
}

onMounted(() => {
  let loginInfo = userStore.loginInfo
  if (loginInfo) {
    rbPassword.value = true
    formData.account = loginInfo.account
    formData.password = loginInfo.password
  }
  getValidateCode()
})
</script>


<style lang="scss" scoped>
.container {
  height: 100%;
  position: relative;
  overflow: hidden;
  background: linear-gradient(217deg, rgba(113, 255, 113, 0.8), rgba(113, 255, 113, 0) 70.71%),
    linear-gradient(127deg, rgba(77, 174, 255, 0.8), rgba(77, 174, 255, 0) 70.71%),
    linear-gradient(336deg, rgba(3, 249, 249, 0.8), rgba(3, 249, 249, 0) 70.71%);
}

.g-bg {
  position: absolute;
  height: 100%;
  width: 100%;
  z-index: 1;

  &::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    backdrop-filter: blur(100px);
    z-index: 1;
  }

  .g-polygon {
    position: absolute;
    opacity: 0.5;

    &-1 {
      width: 60%;
      height: 80%;
      position: absolute;
      left: 50%;
      top: 40%;
      transform: translate(-50%, -50%);
      background: #71ff71;
      clip-path: polygon(0 10%, 30% 0, 100% 40%, 70% 100%, 20% 90%);
    }

    &-2 {
      width: 70%;
      height: 50%;
      position: absolute;
      left: 30%;
      top: 70%;
      transform: translate(-50%, -50%);
      background: #4daeff;
      clip-path: polygon(10% 0, 100% 70%, 100% 100%, 20% 90%);
    }

    &-3 {
      width: 70%;
      height: 50%;
      position: absolute;
      left: 64%;
      top: 70%;
      transform: translate(-50%, -50%);
      background: #03f9f9;
      clip-path: polygon(80% 0, 100% 70%, 100% 100%, 20% 90%);
    }
  }
}

.context {
  position: relative;
  height: 100%;
  z-index: 2;

  .login {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    .title {
      font-size: 30px;
      font-weight: 500;
      margin-top: 7px;

      &-top {
        margin: 0;
      }
    }

    .el-form {
      margin: 25px 0 20px;

      // 图片验证码
      .img-code {
        width: 150px;
        height: 48px;
        margin-left: 20px;
        cursor: pointer;
      }

      .el-button {
        width: 100%;
      }
    }
  }
}
</style>
