<template>
  <el-card shadow="hover" class="todo-item">
    <div v-if="!isEditing" class="todo-content">
      <div class="todo-main">
        <el-checkbox
          :model-value="isDone"
          @change="$emit('checkbox-changed')"
        >
          <span :class="{ 'done': isDone }">{{ label }}</span>
        </el-checkbox>
        <el-tag size="small" type="info">{{ formatDate(date) }}</el-tag>
      </div>
      
      <div class="todo-actions">
        <el-button-group>
          <el-button
            type="primary"
            :icon="Edit"
            size="small"
            ref="editButton"
            @click="toggleToItemEditForm"
          >
            编辑
          </el-button>
          <el-button
            type="danger"
            :icon="Delete"
            size="small"
            @click="$emit('item-deleted')"
          >
            删除
          </el-button>
        </el-button-group>
      </div>
    </div>
    
    <to-do-item-edit-form
      v-else
      :id="id"
      :label="label"
      :date="date"
      @item-edited="itemEdited"
      @edit-cancelled="editCancelled"
      ref="editForm"
    ></to-do-item-edit-form>
  </el-card>
</template>

<script>
import ToDoItemEditForm from "./ToDoItemEditForm.vue"
import { Edit, Delete } from '@element-plus/icons-vue'

export default {
  name: 'ToDoItem',
  components: {
    ToDoItemEditForm
  },
  props: {
    label: { required: true, type: String },
    done: { default: false, type: Boolean },
    id: { required: true, type: String },
    date: { required: true, type: String }
  },
  data() {
    return {
      isEditing: false,
      editText: this.label,
      Edit,
      Delete
    }
  },
  computed: {
    isDone() {
      return this.done
    }
  },
  methods: {
    toggleToItemEditForm() {
      this.isEditing = true
    },
    itemEdited(newData) {
      this.$emit('item-edited', newData)
      this.isEditing = false
    },
    editCancelled() {
      this.isEditing = false
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('zh-CN', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit'
      }).replace(/\//g, '-')
    }
  }
}
</script>

<style scoped>
.todo-item {
  margin-bottom: 10px;
}

.todo-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.todo-main {
  display: flex;
  align-items: center;
  gap: 10px;
}

.todo-actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.done {
  text-decoration: line-through;
  color: var(--el-text-color-secondary);
}

.el-tag {
  margin-left: 10px;
}
</style> 