<template>
  <v-container fluid>
    <v-row>
      <v-col cols="12">
        <v-card v-if="user" class="mb-2">
          <v-card-title>
            <span class="headline font-weight-black">Foydalanuvchi ma'lumotlari</span>
          </v-card-title>
          <v-card-text>
            <p class="text-h6">
              <span class="font-weight-bold">Ismi: </span>{{ user[0].full_name }}
            </p>
            <p class="text-h6">
              <span class="font-weight-bold">Ishlash joyi: </span>
              <span>{{ user[0].branch_name }}. </span>
              <span>{{ user[0].structure_name }} - </span>
              <span>{{ user[0].position_name }}</span>
            </p>
          </v-card-text>
        </v-card>
        <v-card>
          <v-card-title>
            <span class="ml-4 pb-5">Lavozim majburiyatlari formasini to'ldiring</span>
          </v-card-title>
          <v-card-text>
            <v-row>
              <v-col cols="6" sm="4" md="2">
                <v-menu
                  v-model="dateMenu"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="date"
                      label="Sanani tanlang"
                      prepend-icon="mdi-calendar"
                      readonly
                      outlined
                      dense
                      v-bind="attrs"
                      v-on="on"
                      class="ml-4"
                    />
                  </template>
                  <v-date-picker v-model="date" @input="dateMenu = false" :first-day-of-week="1" />
                </v-menu>
              </v-col>
            </v-row>
            <v-form @submit.prevent="sendApplication" v-model="valid" ref="form" v-if="user">
              <v-card flat>
                <v-card-text>
                  <v-row v-for="(task, index) in completedTasks" :key="task.id">
                    <v-col cols="12" md="6">
                      <v-autocomplete
                        v-model="task.task_id"
                        :items="user[1]"
                        :rules="requiredRules"
                        item-value="id"
                        item-text="task_name"
                        outlined
                        dense
                        placeholder="Majburiyat nomi"
                        hint="Majburiyat nomini tanlang"
                        required
                      >
                      </v-autocomplete>
                    </v-col>
                    <v-col cols="12" md="1">
                      <v-text-field
                        v-model="task.quantity"
                        :rules="requiredRules"
                        type="number"
                        outlined
                        dense
                        placeholder="Soni"
                        hint="Sonini kiriting"
                        required
                      />
                    </v-col>
                    <v-col cols="12" md="1">
                      <v-text-field
                        type="number"
                        v-model="task.hour"
                        :rules="hourRules"
                        outlined
                        dense
                        placeholder="Soat"
                        hint=""
                        required
                      />
                    </v-col>
                    <v-col cols="12" md="1">
                      <v-text-field
                        type="number"
                        v-model="task.minute"
                        :rules="minuteRules"
                        outlined
                        dense
                        placeholder="Daqiqa"
                        hint=""
                        required
                      />
                    </v-col>
                    <v-col cols="12" :md="completedTasks.length > 1 ? 2 : 3">
                      <v-textarea
                        v-model="task.comment"
                        rows="1"
                        outlined
                        dense
                        placeholder="Qo'shimcha ma'lumot"
                        hint="Qo'shimcha ma'lumot"
                        auto-grow
                        counter="100"
                      />
                    </v-col>
                    <v-col v-if="completedTasks.length > 1" cols="12" md="1">
                      <v-btn @click.prevent="removeTask(index)" icon color="error" class="ml-5">
                        <v-icon>mdi-close</v-icon>
                      </v-btn>
                    </v-col>
                    <v-divider v-if="completedTasks.length > 1" />
                  </v-row>
                  <v-row>
                    <v-col cols="12" md="6">
                      <v-btn
                        v-if="user[1].length > completedTasks.length"
                        color="primary"
                        outlined
                        @click.prevent="addTask"
                      >
                        Qo'shish
                      </v-btn>
                    </v-col>
                  </v-row>
                </v-card-text>
                <v-card-actions>
                  <v-spacer />
                  <p>
                    <span>Umumiy sarflangan vaqt: </span>{{ totalSpentTime.days }} kun,
                    {{ totalSpentTime.hours }} soat, {{ totalSpentTime.minutes }} daqiqa
                  </p>
                </v-card-actions>
                <v-card-actions>
                  <v-spacer />
                  <v-btn
                    class="mt-6"
                    color="primary"
                    type="submit"
                    @click.prevent="sendApplication"
                    :disabled="!valid"
                  >
                    Arizani jo'natish
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-form>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <v-snackbar v-model="showSnackbar" :color="status">
      {{ message }}
    </v-snackbar>
  </v-container>
</template>

<script>
import { mapGetters } from 'vuex'
export default {
  name: 'AddReportView',
  data() {
    return {
      valid: false,
      department: null,
      position: null,
      dateMenu: false,
      date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
        .toISOString()
        .substr(0, 10),
      completedTasks: [
        {
          task_id: null,
          quantity: null,
          hour: null,
          minute: null,
          comment: '',
        },
      ],
      requiredRules: [(v) => !!v || "To'ldirilishi kerak"],
      hourRules: [
        (v) => !!v || "To'ldirilishi kerak",
        (v) => (v >= 0 && v <= 23) || "Noto'g'ri kiritildi (0-23)",
      ],
      minuteRules: [
        (v) => !!v || "To'ldirilishi kerak",
        (v) => (v >= 0 && v <= 59) || "Noto'g'ri kiritildi (0-59)",
      ],
      showSnackbar: null,
      message: null,
      status: null,
    }
  },
  computed: {
    ...mapGetters('office', ['departments', 'departmentPositions', 'positionTasks']),
    ...mapGetters('auth', ['user']),
    totalSpentTime() {
      let total = this.completedTasks.reduce((acc, task) => {
        return acc + Number(task.hour) * 60 + Number(task.minute)
      }, 0)
      const days = Math.floor(total / (60 * 24))
      total -= days * (60 * 24)
      const hours = Math.floor(total / 60)
      total -= hours * 60
      const minutes = total
      return { days, hours, minutes }
    },
  },
  methods: {
    sendApplication() {
      this.$store
        .dispatch('office/sendApplication', {
          date: this.date,
          tasks: this.completedTasks,
        })
        .then((response) => {
          if (response.result === 'OK') {
            this.showSnackbar = true
            this.status = 'success'
            this.message = "Ariza muvaffaqiyatli jo'natildi"
            this.completedTasks = [
              {
                task_id: null,
                quantity: null,
                hour: null,
                minute: null,
                comment: '',
              },
            ]
            this.$refs.form.reset()
          }
        })
        .catch(() => {
          this.showSnackbar = true
          this.status = 'error'
          this.message = "Ariza jo'natishda xatolik yuz berdi"
        })
    },
    addTask() {
      this.completedTasks.push({
        task_id: null,
        quantity: null,
        hour: null,
        minute: null,
        comment: '',
      })
    },
    removeTask(index) {
      this.completedTasks.splice(index, 1)
    },
  },
  created() {
    this.$store.dispatch('auth/fetchUserData')
  },
}
</script>

<style scoped></style>
