<template>
  <div class="css-calculate-container" v-show="is_show">
    <v-text-field
      :rounded="true"
      class="css-operation-sum"
      readonly
      :value="text_number_display"></v-text-field>
    <v-text-field
      :maxlength="COMMENT_MAX_LENGTH"
      :rounded="true"
      :value="text_operation"
      class="css-operation-comment"
      placeholder="Комментарий"
      v-model="text_operation"
      prepend-inner-icon="mdi-lead-pencil"></v-text-field>
    <div class="css-calculate-grid">
      <button class="css-calculate-button" @click="numberSet(7)">7</button>
      <button class="css-calculate-button" @click="numberSet(8)">8</button>
      <button class="css-calculate-button" @click="numberSet(9)">9</button>
      <button class="css-calculate-button css-calculate-button-date">
        <v-dialog ref="dialog" v-model="is_calender_open">
          <template v-slot:activator="{on, attrs}">
            <div v-bind="attrs" v-on="on">
              <div class="css-date-month-operation">
                {{o_date.i_day}}/{{o_date.i_month}}</div>
              <div class="css-year-operation">{{o_date.i_year}}</div>
            </div>
          </template>
        <v-date-picker locale="ru-RU" v-model="dl_date" @input="dateSelect"></v-date-picker>
        </v-dialog>
      </button>
      <button class="css-calculate-button" @click="numberSet(4)">4</button>
      <button class="css-calculate-button" @click="numberSet(5)">5</button>
      <button class="css-calculate-button" @click="numberSet(6)">6</button>
      <button class="css-calculate-button" @click="operatorSet('+')">
        <v-icon>mdi-plus</v-icon>
      </button>
      <button class="css-calculate-button" @click="numberSet(1)">1</button>
      <button class="css-calculate-button" @click="numberSet(2)">2</button>
      <button class="css-calculate-button" @click="numberSet(3)">3</button>
      <button class="css-calculate-button" @click="operatorSet('-')">
        <v-icon>mdi-minus</v-icon>
      </button>
      <button class="css-calculate-button" @click="numberSet('.')">.</button>
      <button class="css-calculate-button" @click="numberSet(0)">0</button>
      <button class="css-calculate-button" @click="clear()">
        <v-icon>mdi-backspace-outline</v-icon>
      </button>
      <div class="css-calculate-button css-button-save" @click="calculate()" v-if="text_operator">
        <v-icon>mdi-equal</v-icon>
      </div>
      <button
          v-else
          :class="{'blue-grey lighten-5': is_saving}"
          class="css-calculate-button css-button-save"
          @click="save()"
      >
        <v-icon>mdi-check-circle-outline</v-icon>
      </button>
    </div>
    <v-snackbar
      :absolute="true"
      :timeout="2000"
      class="mb-16"
      transition="scroll-y-reverse-transition"
      v-model="is_sum_zero"
    >
      Введите сумму операции
    </v-snackbar>
  </div>
</template>

<script>
import {CoreDate} from '/src/date/CoreDate.js'

export default {
  props: {
    is_show: {
      type: Boolean,
      required: true,
    },
    dl_operation: {
      type: String,
      required: true,
    },
    m_sum: {
      type: Number,
      required: false,
    },
    text_comment: {
      type: String,
      required: false,
    }
  },

  data() {
    return {
      COMMENT_MAX_LENGTH: 60,
      dl_date: this.dl_operation,
      f_sum_operation: 0,
      is_calender_open: false,
      is_sum_zero: false,
      o_date: this.dateObject(this.dl_operation),
      text_operation: this.text_comment ?? '',
      text_number_a: this.m_sum ? this.m_sum.toString() : '',
      text_number_b: '',
      text_number_display: this.m_sum.toString() ?? 0,
      text_operator: null,
      is_saving: 0,
    }
  },

  methods: {
    dateObject(dl_date) {
      const a_date = CoreDate.toArray(dl_date);
      return {
        'i_day': a_date[2],
        'i_month': a_date[1],
        'i_year': a_date[0]
      }
    },

    dateSelect(dl_date) {
      this.o_date = this.dateObject(dl_date)
      this.dl_date = dl_date
      this.is_calender_open = false
    },
    
    operatorSet(operator) {
      if(this.text_operator)
        this.calculate()

      this.text_operator = operator
      this.text_number_display += operator
    },

    numberValidate(text_current_number, i_new_number) {
      const text_number = text_current_number+i_new_number
      const regex = /^([0-9]|[1-9][0-9]{1,5})(\.|\.\d{1,2}?)?$/;
      return text_number.match(regex) !== null
    },

    numberSet(i_number) {
      if(this.text_number_a === '0' && i_number>0)
        this.text_number_a = '';

      if(this.text_operator === null){
        if(!this.numberValidate(this.text_number_a, i_number))
          return;

        this.text_number_a += i_number;
        this.text_number_display = this.text_number_a;
      }
      else {
        if(!this.numberValidate(this.text_number_b, i_number))
          return;

        this.text_number_b += i_number;
        this.text_number_display += i_number;
      }
    },

    calculate() {
      const f_number_a = this.text_number_a ? parseFloat(this.text_number_a) : 0;
      const f_number_b = this.text_number_b ? parseFloat(this.text_number_b) : 0;
      switch(this.text_operator){
        case '+':
          this.f_sum_operation = f_number_a + f_number_b;
          break;
        case '-':
          this.f_sum_operation = f_number_a - f_number_b;
          break;
        case '*':
          this.f_sum_operation = f_number_a * f_number_b;
          break;
        case '/':
          this.f_sum_operation = f_number_a / f_number_b;
          break;
      }
      this.text_number_display = this.f_sum_operation.toString();
      if(this.text_number_a || this.f_sum_operation !== 0)
        this.text_number_a = '' + this.f_sum_operation + '';
      this.text_number_b = '';
      this.text_operator = null;
      this.is_sum_zero = false
    },

    clear(){
      if(!this.text_number_display)
        return;

      const i_current_number_length = this.text_number_display.length;
      const last_char = this.text_number_display.substring(i_current_number_length - 1, i_current_number_length)

      if(last_char === this.text_operator)
        this.text_operator = null
      else if(this.text_number_b)
        this.text_number_b = this.text_number_b.substring(0, this.text_number_b.length - 1)
      else
        this.text_number_a = this.text_number_a.substring(0, this.text_number_a.length - 1)

      if(i_current_number_length === 1) {
        this.text_number_display = 0
        this.f_sum_operation = 0
      }
      else {
        this.text_number_display = this.text_number_display.substring(0, i_current_number_length - 1)
      }
    },

    save() {
      if(this.is_saving)
        return true

      let m_sum = 0;
      if(this.f_sum_operation)
        m_sum = this.f_sum_operation
      else if(this.text_number_a)
        m_sum = this.text_number_a

      if(m_sum === 0)
      {
        this.is_sum_zero = true
        return true;
      }

      m_sum = parseFloat(m_sum);
      this.is_saving = 1

      this.$emit('operation_save', {
        'dl_operation': this.dl_date,
        'm_sum': m_sum,
        'text_comment': this.text_operation
      })
    }
  }
}
</script>

<style lang="scss">
.css-calculate-container {
  background-color: #fbf8f8;
  bottom: 0;
  border-top: 1px solid #CCCCCC;
  position: absolute;
  width: 100%;
}

.css-calculate-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  justify-content: center;
  grid-auto-rows: 50px;
  border-top: 1px solid #CCCCCC;
  border-left: 1px solid #CCCCCC;

  .css-calculate-button {
    border-bottom: 1px solid #CCCCCC;
    border-right: 1px solid #CCCCCC;
    text-align: center;
    background-color: #fbf8f8;

    &.css-calculate-button-date {
      padding-top: 10px !important;
    }
  }

  .css-calculate-button:active {
    background-color: #d0d0d0;
  }

  .css-calculate-button-date {
    .css-date-month-operation {
      font-size: 13px;
    }
    .css-year-operation {
      font-size: 11px;
    }
  }

  .css-button-save {
    background-color: #FFF84D;
    border: 1px solid #FFF84D;
  }

  .css-button-save:active {
    background-color: #fcf95c !important;
  }
}

.css-operation-sum {
  position: absolute;
  padding-top: 5px !important;
  width: 100%;
  text-align: right;
  border-bottom: 0!important;

  .v-input__slot {
    padding-right: 10px!important;
  }

  input {
    text-align: right;
  }
}

.css-operation-comment {
  padding-top: 5px !important;
  border-radius: 0;
  height: 45px;
  width: 75%;
  border-bottom: 0!important;
}

</style>