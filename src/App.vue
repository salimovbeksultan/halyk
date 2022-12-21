<script lang="ts">
import { defineComponent, ref } from "vue";
import { xml2json } from "xml-js";

export default defineComponent({
  data() {
    return {
      iin: "",
      found: null as any,
      list: [{ iin: { _text: "" } }],
    };
  },
  methods: {
    formatIIN(event: any) {
      if (event.target.value.length > this.iin.length) {
        this.iin = event.target.value;
        if (
          this.iin.length === 3 ||
          this.iin.length === 7 ||
          this.iin.length === 11
        ) {
          this.iin += " ";
        }
      } else {
        this.iin = event.target.value;
      }
    },
    search() {
      let found = this.list.filter((el) => el.iin._text === this.iinShortened);
      if (found.length > 0) this.found = true;
      else this.found = false;
    },
  },
  computed: {
    iinShortened(): string {
      return this.iin.replace(/\s/g, "");
    },
    response() {
      if (this.found) {
        return [
          "Клиент обнаружен",
          "Проведение сделок с клиентом не рекомендуется",
        ];
      } else {
        return [
          "Клиент не обнаружен",
          "Проведение сделок с данным клиентом возможно",
        ];
      }
    },
    buttonText() {
      if (this.found != null) {
        return "Выполнить новый поиск";
      } else {
        return "Выполнить поиск";
      }
    },
  },
  mounted() {
    let xmlHttp = new XMLHttpRequest();
    const $self = this;
    xmlHttp.onreadystatechange = function () {
      if (this.readyState == 4 && this.status == 200) {
        let temp = JSON.parse(xml2json(this.response, { compact: true }));
        $self.list = [...temp.xml.persons.person];
      }
    };
    xmlHttp.open("GET", "src/assets/afmr.xml", true);
    xmlHttp.send();
  },
});
</script>

<script setup lang="ts">
const camera = ref();
</script>

<template>
  <main>
    <div class="text-header">
      <span>Поиск клиента в АФМ</span>
    </div>
    <div class="input-group">
      <input
        class="iin-input"
        placeholder="Введите ИИН"
        type="text"
        maxlength="15"
        :value="iin"
        @input="formatIIN"
      />
      <button @click="camera.click()" class="camera-button">
        <img class="button-icon" alt="camera" src="@/assets/camera.svg" />
      </button>
      <input ref="camera" class="hidden" type="file" capture="environment" />
    </div>
    <div class="search-group">
      <button
        :disabled="iin.length != 15"
        @click="search"
        class="search-button"
      >
        {{ buttonText }}
      </button>
    </div>
    <div v-if="found != null" class="client-block">
      <p :class="found ? 'found-text' : 'not-found-text'">
        {{ response[0] }}
      </p>
      <p>{{ response[1] }}</p>
    </div>
    <div class="description-block">
      <span>
        Сервис позволяет найти клиента в списке агенства финансового мониторинга
      </span>
    </div>
  </main>
</template>
