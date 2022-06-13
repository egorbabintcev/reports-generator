<template>
  <div id="app">
    <div class="app__plug">
      <p class="app__plug-title">
        Отображение на экранах такого размера пока не поддержано, извините(
      </p>
    </div>

    <div class="app__header">
      <h1 class="app__header-title">
        TC Reports Generator
        <span class="app__header-subtitle"
          >(Автор:
          <span class="app__header-secret-text"
            >некто
            <a
              class="app__header-secret-link"
              target="_blank"
              href="https://www.youtube.com/watch?v=dQw4w9WgXcQ"
              >Егор Бабинцев</a
            ></span
          >)</span
        >
      </h1>
    </div>

    <div class="app__container">
      <div class="section-32 column">
        <div class="container">
          <div class="section-16">
            <div class="row">
              <input v-model="durationTitle" type="text" class="textfield" />
            </div>

            <div class="row">
              <div class="label">
                <p class="subtitle">Время прихода:</p>

                <input
                  @input="startTime = $event.target.value"
                  :value="timeStringFormatter(startTime)"
                  type="text"
                  class="textfield"
                />
              </div>

              <div class="label">
                <p class="subtitle">Время ухода:</p>

                <input
                  @input="endTime = $event.target.value"
                  :value="timeStringFormatter(endTime)"
                  type="text"
                  class="textfield"
                />
              </div>

              <div class="label">
                <p class="subtitle">Перерыв:</p>

                <input
                  @input="breakTime = $event.target.value"
                  :value="timeStringFormatter(breakTime)"
                  type="text"
                  class="textfield"
                />
              </div>
            </div>
          </div>
        </div>

        <div class="section-16">
          <div class="container">
            <div class="label">
              <input v-model="resultsTitle" type="text" class="textfield" />
            </div>
          </div>

          <div class="label">
            <textarea v-model="results" class="textfield" />
          </div>
        </div>

        <div class="section-16">
          <div class="container">
            <div class="label">
              <input v-model="plansTitle" type="text" class="textfield" />
            </div>
          </div>

          <div class="label">
            <textarea v-model="plans" class="textfield" />
          </div>
        </div>
      </div>

      <div class="section-32 column">
        <div class="container">
          <div class="label">
            <p class="subtitle">Тема письма:</p>

            <input
              @focus="$refs.letterSubject.select()"
              :value="`ОТЧЕТ О РАБОТЕ ЗА ДЕНЬ ${currentTime}`"
              type="text"
              class="textfield"
              ref="letterSubject"
              readonly
            />
          </div>
        </div>

        <div class="label">
          <p class="subtitle">Тело письма:</p>

          <textarea
            v-model="output"
            @focus="$refs.letterBody.select()"
            class="textfield"
            ref="letterBody"
            readonly
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import dayjs from "dayjs";

export default {
  name: "App",
  methods: {
    timeStringFormatter(value) {
      value = value.replace(/\D/g, "");
      let hours = value.slice(0, 2);
      let minutes = value.slice(2, 4);

      if (hours > 23) hours = 23;
      if (minutes > 59) minutes = 59;

      if (value.length > 2) value = `${hours}:${minutes}`;
      if (value.length > 5) value = value.slice(0, 5);

      return value;
    },
    getDateFromString(value) {
      const [hrs, min] = value.split(":");

      const date = new Date(0);

      date.setUTCMinutes(min);
      date.setUTCHours(hrs);

      return date;
    },
  },
  computed: {
    currentTime() {
      return dayjs().format("DD.MM.YYYY");
    },
    startTime: {
      get() {
        return this.timeStringFormatter(this.start);
      },
      set(value) {
        localStorage.setItem("TrueConfReportsGeneratorStartTime", value);
        this.start = value;
      },
    },
    endTime: {
      get() {
        return this.timeStringFormatter(this.end);
      },
      set(value) {
        localStorage.setItem("TrueConfReportsGeneratorEndTime", value);
        this.end = value;
      },
    },
    breakTime: {
      get() {
        return this.timeStringFormatter(this.break);
      },
      set(value) {
        localStorage.setItem("TrueConfReportsGeneratorBreakTime", value);
        this.break = value;
      },
    },
    workTime() {
      const diff = new Date(
        this.getDateFromString(this.endTime) -
          this.getDateFromString(this.startTime) -
          this.getDateFromString(this.breakTime)
      );

      if (diff < 0 || Number.isNaN(+diff)) return ``;

      const hrs = String(diff.getUTCHours());
      const min = String(diff.getUTCMinutes());

      return `${hrs.padStart(2, "0")}:${min.padStart(2, "0")}`;
    },
    output() {
      const strings = [];

      // date and time
      if (this.durationTitle) {
        strings.push(this.durationTitle);
      }
      strings.push(`Дата: ${this.currentTime}`);
      strings.push(`Время: ${this.startTime}-${this.endTime}`);
      strings.push(`Перерыв: ${this.breakTime}`);
      strings.push(`Рабочих часов: ${this.workTime}\n`);

      // results
      if (this.results) {
        if (this.resultsTitle) {
          strings.push(this.resultsTitle);
        }
        strings.push(`${this.results}\n`);
      }

      // plans
      if (this.plans) {
        if (this.plansTitle) {
          strings.push(this.plansTitle);
        }
        strings.push(`${this.plans}\n`);
      }

      return strings.join("\n");
    },
  },
  data: () => ({
    durationTitle:
      localStorage.getItem("TrueConfReportsGeneratorDurationTitle") || "",
    start: localStorage.getItem("TrueConfReportsGeneratorStartTime") || "10:00",
    end: localStorage.getItem("TrueConfReportsGeneratorEndTime") || "19:00",
    break: localStorage.getItem("TrueConfReportsGeneratorBreakTime") || "01:00",

    resultsTitle:
      localStorage.getItem("TrueConfReportsGeneratorResultsTitle") ||
      "Результаты:",
    results: localStorage.getItem("TrueConfReportsGeneratorResults") || "",

    plansTitle:
      localStorage.getItem("TrueConfReportsGeneratorPlansTitle") || "Планы:",
    plans: localStorage.getItem("TrueConfReportsGeneratorPlans") || "",
  }),
  watch: {
    durationTitle(value) {
      localStorage.setItem("TrueConfReportsGeneratorDurationTitle", value);
    },
    resultsTitle(value) {
      localStorage.setItem("TrueConfReportsGeneratorResultsTitle", value);
    },
    results(value) {
      localStorage.setItem("TrueConfReportsGeneratorResults", value);
    },
    plansTitle(value) {
      localStorage.setItem("TrueConfReportsGeneratorPlansTitle", value);
    },
    plans(value) {
      localStorage.setItem("TrueConfReportsGeneratorPlans", value);
    },
  },
};
</script>

<style lang="scss">
@use "~reset-css/sass/reset";
@use "./styles/mixins" as *;
@use "./styles/section" as *;

* {
  box-sizing: border-box;
}

textarea {
  min-height: 200px;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

  display: flex;
  flex-flow: column nowrap;

  height: 100vh;
  padding: 50px 40px 40px;
}

.app__plug {
  display: flex;
  flex-flow: column nowrap;
  align-items: center;
  justify-content: center;
  flex-grow: 1;

  @media screen and (min-width: 1024px) {
    display: none;
  }
}

.app__plug-title {
  color: rgba(42 50 58 / 87%);
  font-size: 32px;
  font-weight: 700;
  font-family: monospace;
  line-height: 1.3;
}

.app__container {
  display: flex;
  flex-flow: row nowrap;
  flex-grow: 1;

  max-width: 1170px;
  width: 100%;
  margin: 0 auto;

  @include between-children() {
    margin-right: 32px;
  }

  @media screen and (max-width: 1023px) {
    display: none;
  }
}

.app__header {
  text-align: left;

  margin-bottom: 12px;

  position: fixed;
  top: 8px;
  left: 50%;
  transform: translateX(-50%);

  @media screen and (max-width: 1023px) {
    display: none;
  }
}

.app__header-title {
  color: rgba(42 50 58 / 87%);
  font-size: 18px;
  font-weight: 700;
  font-family: monospace;
  line-height: 1.3;
  white-space: nowrap;
}

.app__header-subtitle {
  color: rgba(42 50 58 / 87%);
  font-size: 12px;
  font-weight: 500;
  font-family: monospace;
  line-height: 1.3;
}

.app__header-secret-text {
  background-color: rgba(42 50 58 / 100%);

  cursor: pointer;

  transition: background-color 0.2s;

  &:hover {
    background-color: transparent;
  }
}

.app__header-secret-link {
  color: rgba(42 50 58 / 87%);
}

.subtitle {
  color: rgba(0 0 0 / 87%);
  font-size: 18px;
  font-weight: 500;
  font-family: monospace;
  line-height: 1.3;
  letter-spacing: 80%;
}

.textfield {
  color: rgba(0 0 0 / 87%);
  font-size: 18px;
  font-family: monospace;
  line-height: 1.3;

  flex-grow: 1;

  resize: none;
  padding: 6px;
  width: 100%;

  @include trueconf-scrollbar();
}

.column {
  flex: 1 0 0;
}

.label {
  display: flex;
  flex-flow: column nowrap;
  align-items: flex-start;
  flex-grow: 1;

  @include between-children() {
    margin-bottom: 8px;
  }
}

.label--shrink {
  flex-grow: 0;
  flex-shrink: 1;
}

.row {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;

  @include between-children() {
    margin-right: 12px;
  }
}

.row--grow {
  flex-grow: 1;
}
</style>
