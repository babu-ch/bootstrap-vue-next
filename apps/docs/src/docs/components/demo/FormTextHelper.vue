<template>
  <!-- #region template -->
  <BForm @submit.stop.prevent>
    <label for="text-password">Password</label>
    <BFormInput
      id="text-password"
      type="password"
      aria-describedby="password-help-block"
    />
    <BFormText id="password-help-block">
      Your password must be 8-20 characters long, contain letters and numbers, and must not contain
      spaces, special characters, or emoji.
    </BFormText>
  </BForm>
  <!-- #endregion template -->
</template>
