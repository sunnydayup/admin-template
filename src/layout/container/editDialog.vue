<template>
  <el-dialog class="dialog-form" width="600px" :visible.sync="dialogFormVisible" @close="$emit('update:show', false)">
    <div slot="title" class="tc title">{{title}}</div>
    <el-form :model="form" ref="form" v-if="dialogFormVisible" label-width="130px">
      <!-- <el-form-item label="请输入原密码：" prop="old_password" :rules="[{required: true,validator:validator_oldpass, trigger: 'blur'}]">
        <el-input v-model="form.old_password" placeholder="请输入原密码" @keyup.enter.native="submitForm('form')" class="w300" size="mini" type="password"></el-input>
      </el-form-item> -->
      <el-form-item label="请输入新密码：" prop="password" :rules="[{required: true,validator:validator_pass, trigger: 'blur'}]">
        <el-input v-model="form.password" placeholder="请输入新密码" @keyup.enter.native="submitForm('form')" class="w300" size="mini" type="password"></el-input>
      </el-form-item>
      <el-form-item label="请确认新密码：" prop="checkPass" :rules="[{required: true,validator:validator_checkPass, trigger: 'blur'}]">
        <el-input v-model="form.checkPass" placeholder="请确认新密码" @keyup.enter.native="submitForm('form')" class="w300" size="mini" type="password"></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="tc">
      <el-button type="primary" class="w100" @click="submitForm('form')">确定</el-button>
      <el-button class="w100" @click="dialogFormVisible =false">取 消</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { postMyPassword } from '@/api/system'
import validate from '@/mixin/validate'
import { mapGetters } from 'vuex'
export default {
  name: 'editDialog',
  mixins: [validate],
  props: {
    show: {
      type: Boolean,
      default: false
    }
  },
  components: {},
  data() {
    return {
      dialogFormVisible: this.show,
      title: '修改密码',
      form: {}
    }
  },
  computed: {
    ...mapGetters(['userInfo'])
  },
  watch: {
    show() {
      this.dialogFormVisible = this.show
    }
  },
  methods: {
    async postMyPassword() {
      await postMyPassword({ password: this.form.password })
      this.$message({
        type: 'success',
        message: '修改成功,请重新登录'
      })
      this.$store.dispatch('logout')
    },
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.postMyPassword()
        } else {
          return false
        }
      })
    },
    validator_oldpass(rule, value, callback) {
      if (value === '' || value === null || value === undefined) {
        callback(new Error('请输入原密码'))
      } else {
        callback()
      }
    },
    validator_pass(rule, value, callback) {
      const reg = /^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,16}$/
      if (value === '' || value === null || value === undefined) {
        callback(new Error('请输入新密码'))
      } else if (!reg.test(value)) {
        callback(new Error('请输入6～16位由数字和字母组成的用户登录密码'))
      } else if (value === this.form.old_password) {
        callback(new Error('新密码与原密码不能一致'))
      } else {
        if (this.form.checkPass) {
          this.$refs.form.validateField('checkPass')
        }
        callback()
      }
    },
    validator_checkPass(rule, value, callback) {
      if (value === '' || value === undefined || value === null) {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.form.password) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }
  },
  created() { },
  mounted() { }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" scoped>
</style>
