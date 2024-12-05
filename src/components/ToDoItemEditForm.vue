<template>
  <el-form @submit.prevent="onSubmit" class="edit-form">
    <el-form-item>
      <el-input
        v-model.trim="newLabel"
        :placeholder="`编辑: ${label}`"
        ref="input"
        @keyup.enter="onSubmit"
        @keyup.esc="onCancel"
      />
    </el-form-item>
    <el-form-item>
      <el-date-picker
        v-model="newDate"
        type="date"
        placeholder="选择日期"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"
        :disabled-date="disabledDate"
        style="width: 100%"
      />
    </el-form-item>
    <el-form-item>
      <el-button-group>
        <el-button @click="onCancel">取消</el-button>
        <el-button type="primary" native-type="submit">保存</el-button>
      </el-button-group>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  props: {
    label: {
      type: String,
      required: true
    },
    id: {
      type: String,
      required: true
    },
    date: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      newLabel: this.label,
      newDate: this.date
    }
  },
  mounted() {
    this.$nextTick(() => {
      if (this.$refs.input) {
        this.$refs.input.focus()
      }
    })
  },
  methods: {
    onSubmit() {
      if (this.newLabel && this.newLabel !== this.label || this.newDate !== this.date) {
        this.$emit('item-edited', {
          label: this.newLabel,
          date: this.newDate
        })
      } else {
        this.$emit('edit-cancelled')
      }
    },
    onCancel() {
      this.$emit('edit-cancelled')
    },
    disabledDate(time) {
      // 可以在这里添加日期限制，比如不能选择未来日期
      return time.getTime() > Date.now()
    }
  }
}
</script>

<style scoped>
.edit-form {
  margin-top: 10px;
}

.el-button-group {
  margin-top: 10px;
}

.el-date-picker {
  width: 100%;
}
</style>
  