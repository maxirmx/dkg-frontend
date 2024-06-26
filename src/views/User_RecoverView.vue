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

import { Form, Field } from 'vee-validate'
import * as Yup from 'yup'
import router from '@/router'
import { useAuthStore } from '@/stores/auth.store.js'
import { useAlertStore } from '@/stores/alert.store.js'

const schema = Yup.object().shape({
  email: Yup.string()
    .required('Please enter an email address')
    .email('Wrong email format')
})

function onSubmit(values, { setErrors }) {
  const authStore = useAuthStore()
  values.host = window.location.href
  values.host = values.host.substring(0, values.host.lastIndexOf('/'))
  return authStore
    .recover(values)
    .then(() => {
      router.push('/').then(() => {
        const alertStore = useAlertStore()
        alertStore.success(
          'A letter with a link to recover your password has been sent to your email address. ' +
          'Please note that the link is one-time use and is valid for 4 hours. ' +
          'If you cannot find the letter, check your junk mail (spam) folder. ' +
          'If the letter did not arrive, please contact the administrator.'        )
      })
    })
    .catch((error) => setErrors({ apiError: error }))
}
</script>

<template>
  <div class="settings form-1">
    <h1 class="orange">Recover password</h1>
    <hr class="hr" />
    <Form @submit="onSubmit" :validation-schema="schema" v-slot="{ errors, isSubmitting }">
      <div class="form-group">
        <label for="email" class="label">Email:</label>
        <Field
          name="email"
          id="email"
          type="text"
          class="form-control input"
          :class="{ 'is-invalid': errors.email }"
          placeholder="Email"
        />
      </div>
      <div class="form-group">
        <button class="button" type="submit" :disabled="isSubmitting">
          <span v-show="isSubmitting" class="spinner-border spinner-border-sm mr-1"></span>
          Send recover password link
        </button>
      </div>
      <div v-if="errors.email" class="alert alert-danger mt-3 mb-0">{{ errors.email }}</div>
      <div v-if="errors.apiError" class="alert alert-danger mt-3 mb-0">{{ errors.apiError }}</div>
    </Form>
  </div>
</template>
