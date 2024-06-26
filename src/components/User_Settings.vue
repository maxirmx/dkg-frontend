<script setup>
// Copyright (C) 2023-2024 Maxim [maxirmx] Samsonov  (www.sw.consulting)
// All rights reserved.
// This file is a part of Dkg Frontend applcation
//
// Redistribution and use in source and binary forms, with or without
// modification, are permitted provided that the following conditions
// are met:
// 1. Redistributions of source code must retain the above copyright
// notice, this list of conditions and the following disclaimer.
// 2. Redistributions in binary form must reproduce the above copyright
// notice, this list of conditions and the following disclaimer in the
// documentation and/or other materials provided with the distribution.
//
// THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
// ``AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED
// TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
// PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDERS OR CONTRIBUTORS
// BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
// CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
// SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
// INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
// CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
// ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
// POSSIBILITY OF SUCH DAMAGE.

import { ref } from 'vue'

import router from '@/router'
import { storeToRefs } from 'pinia'
import { Form, Field } from 'vee-validate'
import * as Yup from 'yup'
import { useUsersStore } from '@/stores/users.store.js'
import { useAuthStore } from '@/stores/auth.store.js'
import { useAlertStore } from '@/stores/alert.store.js'
import { pwdErr, pwdReg} from '@/helpers/pwd.js'

const props = defineProps({
  register: {
    type: Boolean,
    required: true
  },
  id: {
    type: Number,
    required: false
  }
})

const usersStore = useUsersStore()
const authStore = useAuthStore()

const schema = Yup.object().shape({
  name: Yup.string().required('Please enter a (nick)name'),
  email: Yup.string()
    .required('Please enter an email address')
    .email('Wrong email format'),
  password: Yup.string().concat(
    isRegister() ? Yup.string().required('Please enter a password').matches(pwdReg, pwdErr) : null
  ),
  password2: Yup.string()
    .when('password', (password, schema) => {
      if ((password && password != '') || isRegister())
        return schema.required('Please confirm password').matches(pwdReg, pwdErr)
    })
    .oneOf([Yup.ref('password')], 'Passwords shall match')
})

const showPassword = ref(false)
const showPassword2 = ref(false)

let user = ref({
  isEnabled: true,
})

if (!isRegister()) {
  ;({ user } = storeToRefs(usersStore))
  await usersStore.getById(props.id, true)
}

function isRegister() {
  return props.register
}

function asAdmin() {
  return authStore.user && authStore.user.isAdmin
}

function getTitle() {
  return isRegister() ? (asAdmin() ? 'Create new user' : 'Self registration') : 'Settings'
}

function getButton() {
  return isRegister() ? (asAdmin() ? 'Create' : 'Register') : 'Save'
}

function showCredentials() {
  return !isRegister() && !asAdmin()
}

function showAndEditCredentials() {
  return asAdmin()
}

function getCredentials() {
  let crd = null
  if (user.value) {
    crd = ''
    if (user.value.isAdmin === 'ADMIN') {
      crd = 'Admin'
    }
  }
  return crd
}

function onSubmit(values, { setErrors }) {
  if (isRegister()) {
    if (asAdmin()) {
      return usersStore
        .add(values, true)
        .then(() =>
          router.push(authStore.user.isAdmin ? '/users' : '/user/edit/' + authStore.user.id)
        )
        .catch((error) => setErrors({ apiError: error }))
    } else {
      values.isAdmin = false
      values.host = window.location.href
      values.host = values.host.substring(0, values.host.lastIndexOf('/'))
      return authStore
        .register(values)
        .then(() => {
          router.push('/').then(() => {
            const alertStore = useAlertStore()
            alertStore.success(
              'A confirmation email has been sent to your email address. ' +
              'Please follow the link to complete the registration. ' +
              'Please note that the link is one-time and valid for 4 hours. ' +
              'If you can\'t find the email, check your junk mail (spam) folder. ' +
              'If the email didn\'t arrive, please contact the administrator.')
          })
        })
        .catch((error) => setErrors({ apiError: error }))
    }
  } else {
    return usersStore
      .update(props.id, values, true)
      .then(() =>
        router.push(authStore.user.isAdmin ? '/users' : '/user/edit/' + authStore.user.id)
      )
      .catch((error) => {
        console.log(error)
        setErrors({ apiError: error })
      })
  }
}

</script>

<template>
  <div class="settings form-2">
    <h1 class="orange">{{ getTitle() }}</h1>
    <hr class="hr" />
    <Form
      @submit="onSubmit"
      :initial-values="user"
      :validation-schema="schema"
      v-slot="{ errors, isSubmitting }"
    >
      <div class="form-group">
        <label for="name" class="label">(Nick)name:</label>
        <Field
          name="name"
          id="name"
          type="text"
          class="form-control input"
          :class="{ 'is-invalid': errors.name }"
          placeholder="Name"
        />
      </div>
      <div class="form-group">
        <label for="email" class="label">Email:</label>
        <Field
          name="email"
          id="email"
          autocomplete="off"
          type="email"
          class="form-control input"
          :class="{ 'is-invalid': errors.email }"
          placeholder="Email"
        />
      </div>
      <div class="form-group">
        <label for="password" class="label">Password:</label>
        <Field
          name="password"
          id="password"
          ref="password"
          :type="showPassword ? 'text' : 'password'"
          class="form-control input password"
          :class="{ 'is-invalid': errors.password }"
          placeholder="password"
        />
        <button
          type="button"
          @click="
            (event) => {
              event.preventDefault()
              showPassword = !showPassword
            }
          "
          class="button-o"
        >
          <font-awesome-icon
            size="1x"
            v-if="!showPassword"
            icon="fa-solid fa-eye"
            class="button-o-c"
          />
          <font-awesome-icon
            size="1x"
            v-if="showPassword"
            icon="fa-solid fa-eye-slash"
            class="button-o-c"
          />
        </button>
      </div>
      <div class="form-group">
        <label for="password2" class="label">Repeat password:</label>
        <Field
          name="password2"
          id="password2"
          :type="showPassword2 ? 'text' : 'password'"
          class="form-control input password"
          :class="{ 'is-invalid': errors.password2 }"
          placeholder="password"
        />

        <button
          type="button"
          @click="
            (event) => {
              event.preventDefault()
              showPassword2 = !showPassword2
            }
          "
          class="button-o"
        >
          <font-awesome-icon
            size="1x"
            v-if="!showPassword2"
            icon="fa-solid fa-eye"
            class="button-o-c"
          />
          <font-awesome-icon
            size="1x"
            v-if="showPassword2"
            icon="fa-solid fa-eye-slash"
            class="button-o-c"
          />
        </button>
      </div>
      <div v-if="showCredentials()" class="form-group">
        <label for="crd" class="label">Права:</label>
        <span id="crd"
          ><em>{{ getCredentials() }}</em></span
        >
      </div>

      <div v-if="showAndEditCredentials()" class="form-group">
        <label for="isAdmin" class="label">Права:</label>
        <Field
          id="isAdmin"
          type="checkbox"
          name="isAdmin"
          class="checkbox checkbox-styled"
          value="ADMIN"
        />
        <label for="isAdmin">Администратор</label>
      </div>

      <div class="form-group">
        <button class="button" type="submit" :disabled="isSubmitting">
          <span v-show="isSubmitting" class="spinner-border spinner-border-sm mr-1"></span>
          {{ getButton() }}
        </button>
        <button
          v-if="asAdmin()"
          class="button"
          type="button"
          @click="
            $router.push(authStore.user.isAdmin ? '/users' : '/user/edit/' + authStore.user.id)
          "
        >
          <span v-show="isSubmitting" class="spinner-border spinner-border-sm mr-1"></span>
          Отменить
        </button>
      </div>
      <div v-if="errors.name" class="alert alert-danger mt-3 mb-0">{{ errors.name }}</div>
      <div v-if="errors.email" class="alert alert-danger mt-3 mb-0">{{ errors.email }}</div>
      <div v-if="errors.password" class="alert alert-danger mt-3 mb-0">{{ errors.password }}</div>
      <div v-if="errors.password2" class="alert alert-danger mt-3 mb-0">{{ errors.password2 }}</div>
      <div v-if="errors.apiError" class="alert alert-danger mt-3 mb-0">{{ errors.apiError }}</div>
    </Form>
  </div>
  <div v-if="user?.loading" class="text-center m-5">
    <span class="spinner-border spinner-border-lg align-center"></span>
  </div>
  <div v-if="user?.error" class="text-center m-5">
    <div class="text-danger">Failed to load user data: {{ user.error }}</div>
  </div>
</template>
