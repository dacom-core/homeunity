<template>
  <form class="full-width" @submit.prevent="submit">
    <q-input
      v-model="privateKey"
      label="Приватный ключ или фраза"
      hint="Введите приватный ключ или сид фразу"
      outlined
      class="full-width"
      type="password"
      autocorrect="off"
      autocapitalize="off"
      autocomplete="off"
      spellcheck="false" />

    <div class="row justify-end">
      <q-btn
        type="submit"
        label="Войти"
        class="q-mt-md full-width"
        color="teal"
        :loading="loading"
        :disable="!privateKey" />
    </div>
  </form>
</template>

<script setup lang="ts">
  import { ref } from 'vue'
  import { Notify } from 'quasar'
  import { useRouter } from 'vue-router'

  import { isValidWif, makeAccountByWif, makeAccountByMnemonic } from 'unicore'
  import Chains from '~/chainsMain'
  import { useUserStore } from '~/stores/user'

  const privateKey = ref('')
  const loading = ref(false)
  const userStore = useUserStore()
  const router = useRouter()

  const submit = async () => {
    loading.value = true
    try {
      let account
      let isWif = false
      try {
        isWif = isValidWif(privateKey.value)
      } catch (e) {
        // NOOP
      }
      if (isWif) {
        account = await makeAccountByWif('', privateKey.value)
      } else {
        account = await makeAccountByMnemonic('', privateKey.value)
      }

      const rootChain = Chains.getRootChain()
      const data = await rootChain.readApi.getKeyAccounts(account.pub)

      const {
        account_names: [username],
      } = data

      if (username) {
        account.name = username

        await userStore.login(account)
        await router.push({ name: 'lk-estate' })
        Notify.create({
          message: 'Успешный вход',
          type: 'positive',
        })
        loading.value = false
        return
      }
    } catch (e) {
      console.error('COMMON', e)
    }
    Notify.create({
      message: 'Профиль не найден',
      type: 'negative',
    })
    loading.value = false
  }
</script>

<style lang="scss" scoped></style>
