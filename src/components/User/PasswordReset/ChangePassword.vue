<template>
  <div>
    <b-jumbotron>
      <div class="form-group">
        <label for="oldPasswordInput">New password</label>
        <input type="password" class="form-control" id="oldPasswordInput" placeholder="Old password"
               v-model="oldPassword">
      </div>
      <div class="form-group">
        <label for="passwordInput">New password</label>
        <input type="password" class="form-control" id="passwordInput" placeholder="New password" v-model="password">
      </div>
      <div class="form-group">
        <label for="passwordVerificationInput">Confirm new password</label>
        <input type="password" class="form-control" id="passwordVerificationInput" placeholder="Reenter new password"
               v-model="passwordRepeated">
      </div>
      <div class="form-group" v-if="!AreInputsValid || !IsPasswordSafe || !DoPasswordsMatch">
        <p>{{ errorMessage }}</p>
      </div>
      <button type="submit" class="btn btn-primary btn-block" v-on:click="onSubmit()">Submit</button>
    </b-jumbotron>
  </div>
</template>

<script>
export default {
  name: "ChangePassword",
  data: function () {
    return {
      oldPassword: null,
      password: null,
      passwordRepeated: null,
      errorMessage: ''
    }
  },
  methods: {
    onSubmit() {
      if (!this.AreInputsValid) {
        this.errorMessage = 'All fields must be filled!';
        return;
      }

      if (!this.DoPasswordsMatch) {
        this.errorMessage = 'Passwords must match!';
        return;
      }

      if (!this.IsPasswordSafe) {
        this.errorMessage = 'Password must contain at least 10 characters, a lowercase, an uppercase, a number and one special sign';
        return;
      }

      let data = {
        newPassword: this.password,
        newPasswordRepeated: this.passwordRepeated,
        oldPassword: this.oldPassword
      }
      let router = this.$router;

      this.$http
          .post(process.env.VUE_APP_BACKEND_URL + 'register/password/change', data)
          .then(response => {
            response.data
            alert("Password changed successfully! Proceed to log in!")
            this.$store.dispatch('logOut');
            router.push('/login');
          }).catch(err => {
        err.data
        alert("Password change failed. Please make sure the password provided is correct and that the new password is typed correctly both times and has 10 characters, a lowercase, an uppercase, a number and a special sign.")
      });
    },

    dispatch(type) {
      let router = this.$router;
      if (type === 'InstagramUser') {
        router.push('/instagram');
        return
      } else {
        router.push('/admin');
        return
      }
    }
  },
  computed: {
    AreInputsValid() {
      return this.oldPassword !== null && this.password !== null && this.passwordRepeated !== null
          && this.oldPassword !== '' && this.password !== '' && this.passwordRepeated !== '';
    },
    DoPasswordsMatch() {
      return this.password === this.passwordRepeated;
    },
    IsPasswordSafe() {
      if (this.password === null) return false;
      let hasMinimumLength = (this.password.length > 8);
      let hasNumber = /\d/.test(this.password);
      let hasLowercase = /[a-z]/.test(this.password);
      let hasUppercase = /[A-Z]/.test(this.password);
      let hasSpecial = /[!@#$%^&*)(+=._-]/.test(this.password);
      let hasForbiddenChar = /[<>]/.test(this.password) || /[<>]/.test(this.passwordRepeated) || /[<>]/.test(this.oldPassword);
      return hasMinimumLength && hasNumber && hasLowercase && hasUppercase && hasSpecial && !hasForbiddenChar;
    }
  }
}
</script>

<style scoped>

</style>
