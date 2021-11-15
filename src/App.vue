<template>
  <div class="container">
    <app-alert :alert="alert" @close="alert = null"></app-alert>

    <form class="card" @submit.prevent="createPerson">
      <h2>Работа с базой данных</h2>

      <div class="form-control">
        <label for="name">Введите имя</label>
        <input type="text" id="name" v-model.trim="name" />
      </div>

      <button class="btn primary" :disabled="name.length === 0">
        Создать человека
      </button>
    </form>

    <app-loader v-if="loading"></app-loader>

    <app-people-list
      v-else
      :people="people"
      @load="loadPeople"
      @remove="removePerson"
    ></app-people-list>
  </div>
</template>

<script>
import AppPeopleList from './AppPeopleList.vue'
import AppAlert from './AppAlert.vue'
import AppLoader from './AppLoader.vue'
import axios from 'axios'

export default {
  data() {
    return {
      name: '',
      people: [],
      alert: null,
      loading: false,
    }
  },
  mounted() {
    this.loadPeople()
  },
  methods: {
    async createPerson() {
      const response = await fetch(
        'https://vue-with-http-5b58f-default-rtdb.europe-west1.firebasedatabase.app/people.json',
        {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            firstName: this.name,
          }),
        }
      )

      const firebaseData = await response.json()

      this.people.push({
        id: firebaseData.name,
        firstName: this.name,
      })

      this.name = ''
    },
    loadPeople() {
      // запускаем картинку загрузки
      this.loading = true
      // исскуственная задержка
      setTimeout(async () => {
        // обработка ошибок
        try {
          const { data } = await axios.get(
            'https://vue-with-http-5b58f-default-rtdb.europe-west1.firebasedatabase.app/people.json'
          )
          // если данных нет, говорим об этом пользователю
          if (!data) {
            throw new Error('Нет данных')
          }
          // если данные есть, преобразовываем в нужный формат
          this.people = Object.keys(data).map((key) => {
            return {
              id: key,
              ...data[key],
            }
          })
          // если все ок, останавливаем картинку загрузки
          this.loading = false
        } catch (error) {
          // появилась ошибка при загрузки данных
          this.alert = {
            type: 'danger',
            title: 'Ошибка!',
            text: error.message,
          }
          // останавливаем картинку загрузки
          this.loading = false
        }
      }, 1500)
    },
    async removePerson(id) {
      try {
        const name = this.people.find((person) => person.id === id).firstName
        await axios.delete(
          `https://vue-with-http-5b58f-default-rtdb.europe-west1.firebasedatabase.app/people/${id}.json`
        )
        this.people = this.people.filter((person) => person.id !== id)
        this.alert = {
          type: 'primary',
          title: 'Успешно!',
          text: `Пользователь с именем ${name} был удален`,
        }
      } catch (error) {
        this.alert = {
          type: 'danger',
          title: 'Ошибка!',
          text: error.message,
        }
      }
    },
  },
  components: {
    AppPeopleList,
    AppAlert,
    AppLoader,
  },
}
</script>

<style></style>
