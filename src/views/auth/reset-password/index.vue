<template>
  <div class="flex w-full h-screen">
    <LoginLeftView />

    <div class="relative flex-1">
      <AuthTopBar />

      <div class="auth-right-wrap">
        <div class="form">
          <h3 class="title">{{ $t('forgetPassword.title') }}</h3>
          <p class="sub-title">{{ '输入您的新密码来完成密码重置' }}</p>
          <div class="mt-5">
            <ElForm ref="formRef" :model="form" :rules="rules">
              <ElFormItem prop="password">
                <ElInput
                  class="custom-height"
                  v-model.trim="form.password"
                  :placeholder="$t('register.placeholder.password')"
                  type="password"
                  autocomplete="off"
                  show-password
                />
              </ElFormItem>

              <ElFormItem prop="confirmPassword">
                <ElInput
                  class="custom-height"
                  v-model.trim="form.confirmPassword"
                  :placeholder="$t('register.placeholder.confirmPassword')"
                  type="password"
                  autocomplete="off"
                  @keyup.enter="register"
                  show-password
                />
              </ElFormItem>
            </ElForm>
          </div>

          <div class="mt-[15px]">
            <ElButton
              class="w-full custom-height"
              type="primary"
              @click="handleSubmit"
              :loading="loading"
              v-ripple
            >
              {{ $t('forgetPassword.submitBtnText') }}
            </ElButton>
          </div>

          <div class="mt-[15px]">
            <ElButton class="w-full custom-height" plain @click="toLogin">
              {{ $t('forgetPassword.backBtnText') }}
            </ElButton>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import type { FormRules } from 'element-plus'
  import { useI18n } from 'vue-i18n'
  import { register, resetPassword } from '@/api/auth'

  defineOptions({ name: 'ForgetPassword' })

  const { t } = useI18n()
  const router = useRouter()

  const loading = ref(false)

  const formRef = ref<HTMLFormElement | null>(null)
  const form = ref({
    password: '',
    confirmPassword: ''
  })

  const rules = computed<FormRules<{ password: string; confirmPassword: string }>>(() => ({
    password: [
      { required: true, validator: validatePassword, trigger: 'change' },
      { min: 6, message: t('register.rule.passwordLength'), trigger: 'change' }
    ],
    confirmPassword: [{ required: true, validator: validateConfirmPassword, trigger: 'change' }]
  }))

  /**
   * 验证密码
   * 当密码输入后，如果确认密码已填写，则触发确认密码的验证
   */
  const validatePassword = (_rule: any, value: string, callback: (error?: Error) => void) => {
    if (!value) {
      callback(new Error(t('register.placeholder.password')))
      return
    }

    if (form.value.confirmPassword) {
      formRef.value?.validateField('confirmPassword')
    }

    callback()
  }

  /**
   * 验证确认密码
   * 检查确认密码是否与密码一致
   */
  const validateConfirmPassword = (
    _rule: any,
    value: string,
    callback: (error?: Error) => void
  ) => {
    if (!value) {
      callback(new Error(t('register.rule.confirmPasswordRequired')))
      return
    }

    if (value !== form.value.password) {
      callback(new Error(t('register.rule.passwordMismatch')))
      return
    }

    callback()
  }

  const handleSubmit = async () => {
    await formRef.value?.validate()
    const params: Pick<Api.Auth.RegisterParams, 'password'> = {
      password: form.value.password
    }
    const { error } = await resetPassword(params)
    if (!error) {
      ElMessage.success('密码重置成功,请前往登录')
      toLogin()
    }
  }

  const toLogin = () => {
    router.push({ name: 'Login' })
  }
</script>

<style scoped>
  @import '../login/style.css';
</style>
