<template>
  <div class="container">
    <el-card class="app-card">
      <template #header>
        <div class="card-header">
          <h1>待办事项列表</h1>
        </div>
      </template>

      <el-form @submit.prevent="addTodo" class="todo-form">
        <el-form-item>
          <el-input
            v-model.trim="label"
            placeholder="请输入待办事项"
            :prefix-icon="Edit"
            @keyup.enter="addTodo"
          >
            <template #append>
              <el-button type="primary" @click="addTodo">
                <el-icon><Plus /></el-icon>添加
              </el-button>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-date-picker
            v-model="selectedDate"
            type="date"
            placeholder="选择日期"
            format="YYYY-MM-DD"
            value-format="YYYY-MM-DD"
            style="width: 100%"
          />
        </el-form-item>
      </el-form>

      <div class="filters">
        <el-radio-group v-model="currentFilter" size="large">
          <el-radio-button label="all">全部</el-radio-button>
          <el-radio-button label="active">未完成</el-radio-button>
          <el-radio-button label="completed">已完成</el-radio-button>
        </el-radio-group>
      </div>

      <el-alert
        :title="`总共 ${ToDoItems.length} 项，已完成 ${completedCount} 项`"
        type="info"
        class="summary"
        :closable="false"
        center
      />
      
      <el-scrollbar height="400px">
        <ul class="todo-list">
          <li v-for="(item, index) in filteredTodoItems" :key="item.id">
            <to-do-item
              :label="item.label"
              :done="item.done"
              :id="item.id"
              :date="item.date"
              @checkbox-changed="updateDoneStatus(index)"
              @item-deleted="deleteToDo(index)"
              @item-edited="editToDo(index, $event)"
            ></to-do-item>
          </li>
        </ul>
      </el-scrollbar>

      <div class="chart-container">
        <el-divider>统计图表</el-divider>
        <div id="todoChart" style="width: 100%; height: 400px;"></div>
      </div>
    </el-card>
  </div>
</template>

<script>
import ToDoItem from './components/ToDoItem.vue'
import uniqueId from 'lodash.uniqueid'
import * as echarts from 'echarts'
import { Edit, Plus } from '@element-plus/icons-vue'

export default {
  name: 'app',
  components: {
    ToDoItem
  },
  data() {
    return {
      label: '',
      selectedDate: new Date().toISOString().split('T')[0],
      currentFilter: 'all',
      ToDoItems: JSON.parse(localStorage.getItem('todos')) || [
        { 
          id: uniqueId('todo-'), 
          label: '学习 Vue', 
          done: false,
          date: new Date().toISOString().split('T')[0]
        },
        { 
          id: uniqueId('todo-'), 
          label: '用 CLI 创建 Vue 项目', 
          done: true,
          date: new Date().toISOString().split('T')[0]
        },
        { 
          id: uniqueId('todo-'), 
          label: '开心学习', 
          done: true,
          date: new Date().toISOString().split('T')[0]
        },
        { 
          id: uniqueId('todo-'), 
          label: '创建待办事项列表', 
          done: false,
          date: new Date().toISOString().split('T')[0]
        }
      ],
      Edit,
      Plus
    }
  },
  mounted() {
    this.initChart()
  },
  computed: {
    completedCount() {
      return this.ToDoItems.filter(item => item.done).length
    },
    filteredTodoItems() {
      switch (this.currentFilter) {
        case 'active':
          return this.ToDoItems.filter(item => !item.done)
        case 'completed':
          return this.ToDoItems.filter(item => item.done)
        default:
          return this.ToDoItems
      }
    }
  },
  methods: {
    addTodo() {
      if (this.label === '') {
        return
      }
      this.ToDoItems.push({
        id: uniqueId('todo-'),
        label: this.label,
        done: false,
        date: this.selectedDate
      })
      this.label = ''
      this.selectedDate = new Date().toISOString().split('T')[0]
      this.saveTodos()
      this.updateChart()
    },
    updateDoneStatus(index) {
      if (this.ToDoItems[index]) {
        this.ToDoItems[index].done = !this.ToDoItems[index].done
        this.saveTodos()
        this.updateChart()
      }
    },
    deleteToDo(index) {
      this.ToDoItems.splice(index, 1)
      this.saveTodos()
      this.updateChart()
    },
    editToDo(index, newData) {
      if (this.ToDoItems[index]) {
        this.ToDoItems[index] = {
          ...this.ToDoItems[index],
          label: newData.label,
          date: newData.date
        }
        this.saveTodos()
        this.updateChart()
      }
    },
    saveTodos() {
      localStorage.setItem('todos', JSON.stringify(this.ToDoItems))
    },
    initChart() {
      this.chart = echarts.init(document.getElementById('todoChart'))
      this.updateChart()
    },
    updateChart() {
      const dates = [...new Set(this.ToDoItems.map(item => item.date))].sort()
      
      const completedData = dates.map(date => 
        this.ToDoItems.filter(item => item.date === date && item.done).length
      )
      const uncompletedData = dates.map(date => 
        this.ToDoItems.filter(item => item.date === date && !item.done).length
      )

      const option = {
        title: {
          text: '待办事项统计'
        },
        tooltip: {
          trigger: 'axis'
        },
        legend: {
          data: ['已完成', '未完成']
        },
        xAxis: {
          type: 'category',
          data: dates
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            name: '已完成',
            type: 'bar',
            data: completedData,
            color: '#4CAF50'
          },
          {
            name: '未完成',
            type: 'bar',
            data: uncompletedData,
            color: '#FFA726'
          }
        ]
      }

      this.chart.setOption(option)
    }
  },
  watch: {
    ToDoItems: {
      deep: true,
      handler() {
        this.updateChart()
      }
    }
  }
}
</script>

<style>
.container {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: 100vh;
  padding: 20px;
  background-color: var(--el-bg-color-page);
}

.app-card {
  width: 100%;
  max-width: 800px;
}

.card-header {
  display: flex;
  justify-content: center;
  align-items: center;
}

h1 {
  margin: 0;
  color: var(--el-text-color-primary);
  font-size: 24px;
}

.todo-form {
  margin: 20px 0;
}

.filters {
  display: flex;
  justify-content: center;
  margin: 20px 0;
}

.summary {
  margin: 20px 0;
}

.todo-list {
  list-style: none;
  padding: 0;
  margin: 20px 0;
}

.chart-container {
  margin-top: 30px;
}

/* 覆盖一些Element Plus的默认样式 */
.el-card__header {
  padding: 15px 20px;
}

.el-form-item {
  margin-bottom: 0;
}

.todo-form .el-form-item {
  margin-bottom: 15px;
}

.todo-form .el-date-picker {
  width: 100%;
}
</style>
