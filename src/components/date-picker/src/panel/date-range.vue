<template>
  <transition name="slide-up" @after-leave="$emit('dodestroy')">
    <div
      v-show="visible"
      :style="{ width: width + 'px' }"
      class="x-picker-panel x-date-range-picker"
      :class="[{
        'has-sidebar': $slots.sidebar || shortcuts,
        'has-time': showTime
      }, popperClass]">
      <div class="x-picker-panel-body-wrapper">
        <slot name="sidebar" class="x-picker-panel-sidebar"></slot>
        <div class="x-picker-panel-sidebar" v-if="shortcuts">
          <button
            type="button"
            class="x-picker-panel-shortcut"
            v-for="shortcut in shortcuts"
            @click="handleShortcutClick(shortcut)">{{shortcut.text}}</button>
        </div>
        <div class="x-picker-panel-body">
          <div class="x-date-range-picker-time-header" v-if="showTime">
            <span class="x-date-range-picker-editors-wrap">
              <span class="x-date-range-picker-time-picker-wrap">
                <x-input
                  size="small"
                  ref="minInput"
                  :placeholder="t('x.datepicker.startDate')"
                  class="x-date-range-picker-editor"
                  :value="minVisibleDate"
                  @input.native="handleDateInput($event, 'min')"
                  @change.native="handleDateChange($event, 'min')" />
              </span>
              <span class="x-date-range-picker-time-picker-wrap">
                <x-input
                  size="small"
                  :placeholder="t('x.datepicker.startTime')"
                  class="x-date-range-picker-editor"
                  :value="minVisibleTime"
                  @focus="minTimePickerVisible = !minTimePickerVisible"
                  @change.native="handleTimeChange($event, 'min')" />
                <time-picker
                  :picker-width="minPickerWidth"
                  ref="minTimePicker"
                  :date="minDate"
                  @pick="handleMinTimePick"
                  :visible="minTimePickerVisible">
                </time-picker>
              </span>
            </span>
            <span class="x-icon x-icon-chevron-right"></span>
            <span class="x-date-range-picker-editors-wrap is-right">
              <span class="x-date-range-picker-time-picker-wrap">
                <x-input
                  size="small"
                  :placeholder="t('x.datepicker.endDate')"
                  class="x-date-range-picker-editor"
                  :value="maxVisibleDate"
                  :readonly="!minDate"
                  @input.native="handleDateInput($event, 'max')"
                  @change.native="handleDateChange($event, 'max')" />
              </span>
              <span class="x-date-range-picker-time-picker-wrap">
                <x-input
                  size="small"
                  ref="maxInput"
                  :placeholder="t('x.datepicker.endTime')"
                  class="x-date-range-picker-editor"
                  :value="maxVisibleTime"
                  @focus="minDate && (maxTimePickerVisible = !maxTimePickerVisible)"
                  :readonly="!minDate"
                  @change.native="handleTimeChange($event, 'max')" />
                <time-picker
                  :picker-width="maxPickerWidth"
                  ref="maxTimePicker"
                  :date="maxDate"
                  @pick="handleMaxTimePick"
                  :visible="maxTimePickerVisible"></time-picker>
              </span>
            </span>
          </div>
          <div class="x-picker-panel-content x-date-range-picker-content is-left">
            <div class="x-date-range-picker-header">
              <button
                type="button"
                @click="prevYear"
                class="x-picker-panel-icon-btn x-icon x-icon-chevron-d-left"></button>
              <button
                type="button"
                @click="prevMonth"
                class="x-picker-panel-icon-btn x-icon x-icon-chevron-left"></button>
              <div>{{ leftLabel }}</div>
            </div>
            <date-table
              selection-mode="range"
              :date="date"
              :year="leftYear"
              :month="leftMonth"
              :min-date="minDate"
              :max-date="maxDate"
              :range-state="rangeState"
              :disabled-date="disabledDate"
              @changerange="handleChangeRange"
              :first-day-of-week="firstDayOfWeek"
              @pick="handleRangePick">
            </date-table>
          </div>
          <div class="x-picker-panel-content x-date-range-picker-content is-right">
            <div class="x-date-range-picker-header">
              <button
                type="button"
                @click="nextYear"
                class="x-picker-panel-icon-btn x-icon x-icon-chevron-d-right"></button>
              <button
                type="button"
                @click="nextMonth"
                class="x-picker-panel-icon-btn x-icon x-icon-chevron-right"></button>
              <div>{{ rightLabel }}</div>
            </div>
            <date-table
              selection-mode="range"
              :date="rightDate"
              :year="rightYear"
              :month="rightMonth"
              :min-date="minDate"
              :max-date="maxDate"
              :range-state="rangeState"
              :disabled-date="disabledDate"
              @changerange="handleChangeRange"
              :first-day-of-week="firstDayOfWeek"
              @pick="handleRangePick">
            </date-table>
          </div>
        </div>
      </div>
      <div class="x-picker-panel-footer" v-if="showTime">
        <a
          class="x-picker-panel-link-btn"
          @click="handleClear">{{ t('x.datepicker.clear') }}</a>
        <button
          type="button"
          class="x-picker-panel-btn"
          @click="handleConfirm()"
          :disabled="btnDisabled">{{ t('x.datepicker.confirm') }}</button>
      </div>
    </div>
  </transition>
</template>

<script>
import { nextMonth, prevMonth, toDate, formatDate, parseDate } from '../util';
import TimePicker from './time';
import DateTable from '../basic/date-table';
import XInput from '../../../input';
import Config from '../config';

export default {
    mixins: [Config],
    computed: {
        btnDisabled() {
            return !(this.minDate && this.maxDate && !this.selecting);
        },

        leftLabel() {
            return this.date.getFullYear() + ' '
                + this.t('x.datepicker.year') + ' '
                + this.t(`x.datepicker.month${this.date.getMonth() + 1}`);
        },

        rightLabel() {
            return this.rightDate.getFullYear() + ' '
                + this.t('x.datepicker.year') + ' '
                + this.t(`x.datepicker.month${this.rightDate.getMonth() + 1}`);
        },

        leftYear() {
            return this.date.getFullYear();
        },

        leftMonth() {
            return this.date.getMonth();
        },

        rightYear() {
            return this.rightDate.getFullYear();
        },

        rightMonth() {
            return this.rightDate.getMonth();
        },

        minVisibleDate() {
            return this.minDate ? formatDate(this.minDate) : '';
        },

        maxVisibleDate() {
            return (this.maxDate || this.minDate) ? formatDate(this.maxDate || this.minDate) : '';
        },

        minVisibleTime() {
            return this.minDate ? formatDate(this.minDate, 'HH:mm:ss') : '';
        },

        maxVisibleTime() {
            return (this.maxDate || this.minDate) ? formatDate(this.maxDate || this.minDate, 'HH:mm:ss') : '';
        },

        rightDate() {
            const newDate = new Date(this.date);
            const month = newDate.getMonth();
            newDate.setDate(1);

            if (month === 11) {
                newDate.setFullYear(newDate.getFullYear() + 1);
                newDate.setMonth(0);
            }
            else {
                newDate.setMonth(month + 1);
            }
            return newDate;
        }
    },

    data() {
        return {
            popperClass: '',
            minPickerWidth: 0,
            maxPickerWidth: 0,
            date: new Date(),
            minDate: '',
            maxDate: '',
            rangeState: {
                endDate: null,
                selecting: false,
                row: null,
                column: null
            },
            showTime: false,
            shortcuts: '',
            value: '',
            visible: '',
            disabledDate: '',
            firstDayOfWeek: 7,
            minTimePickerVisible: false,
            maxTimePickerVisible: false,
            width: 0
        };
    },

    watch: {
        showTime(val) {
            if (!val) {
                return;
            }
            this.$nextTick(_ => {
                const minInputElm = this.$refs.minInput.$el;
                const maxInputElm = this.$refs.maxInput.$el;
                if (minInputElm) {
                    this.minPickerWidth = minInputElm.getBoundingClientRect().width + 10;
                }
                if (maxInputElm) {
                    this.maxPickerWidth = maxInputElm.getBoundingClientRect().width + 10;
                }
            });
        },

        minDate() {
            this.$nextTick(() => {
                if (this.maxDate && this.maxDate < this.minDate) {
                    const format = 'HH:mm:ss';

                    this.$refs.maxTimePicker.selectableRange = [
                        [
                            parseDate(formatDate(this.minDate, format), format),
                            parseDate('23:59:59', format)
                        ]
                    ];
                }
            });
        },

        minTimePickerVisible(val) {
            if (val) {
                this.$nextTick(() => this.$refs.minTimePicker.ajustScrollTop());
            }
        },

        maxTimePickerVisible(val) {
            if (val) {
                this.$nextTick(() => this.$refs.maxTimePicker.ajustScrollTop());
            }
        },

        value(newVal) {
            if (!newVal) {
                this.minDate = null;
                this.maxDate = null;
            }
            else if (Array.isArray(newVal)) {
                this.minDate = newVal[0] ? toDate(newVal[0]) : null;
                this.maxDate = newVal[1] ? toDate(newVal[1]) : null;
                if (this.minDate) {
                    this.date = new Date(this.minDate);
                }
                this.handleConfirm(true);
            }
        }
    },

    methods: {
        handleClear() {
            this.minDate = null;
            this.maxDate = null;
            this.handleConfirm(false);
        },

        handleDateInput(event, type) {
            const value = event.target.value;
            const parsedValue = parseDate(value, 'yyyy-MM-dd');

            if (parsedValue) {
                if (typeof this.disabledDate === 'function'
                    && this.disabledDate(new Date(parsedValue))) {
                    return;
                }
                const target = new Date(type === 'min' ? this.minDate : this.maxDate);
                if (target) {
                    target.setFullYear(parsedValue.getFullYear());
                    target.setMonth(parsedValue.getMonth());
                    target.setDate(parsedValue.getDate());
                }
            }
        },

        handleChangeRange(val) {
            this.minDate = val.minDate;
            this.maxDate = val.maxDate;
            this.rangeState = val.rangeState;
        },

        handleDateChange(event, type) {
            const value = event.target.value;
            const parsedValue = parseDate(value, 'yyyy-MM-dd');
            if (parsedValue) {
                const target = new Date(type === 'min' ? this.minDate : this.maxDate);
                if (target) {
                    target.setFullYear(parsedValue.getFullYear());
                    target.setMonth(parsedValue.getMonth());
                    target.setDate(parsedValue.getDate());
                }
                if (type === 'min') {
                    if (target < this.maxDate) {
                        this.minDate = new Date(target.getTime());
                    }
                }
                else {
                    if (target > this.minDate) {
                        this.maxDate = new Date(target.getTime());
                        if (this.minDate && this.minDate > this.maxDate) {
                            this.minDate = null;
                        }
                    }
                }
            }
        },

        handleTimeChange(event, type) {
            const value = event.target.value;
            const parsedValue = parseDate(value, 'HH:mm:ss');
            if (parsedValue) {
                const target = new Date(type === 'min' ? this.minDate : this.maxDate);
                if (target) {
                    target.setHours(parsedValue.getHours());
                    target.setMinutes(parsedValue.getMinutes());
                    target.setSeconds(parsedValue.getSeconds());
                }
                if (type === 'min') {
                    if (target < this.maxDate) {
                        this.minDate = new Date(target.getTime());
                    }
                }
                else {
                    if (target > this.minDate) {
                        this.maxDate = new Date(target.getTime());
                    }
                }
                this.$refs[type + 'TimePicker'].value = target;
                this[type + 'TimePickerVisible'] = false;
            }
        },

        handleRangePick(val, close = true) {
            if (this.maxDate === val.maxDate && this.minDate === val.minDate) {
                return;
            }
            this.maxDate = val.maxDate;
            this.minDate = val.minDate;

            if (!close || this.showTime) {
                return;
            }
            this.handleConfirm();
        },

        changeToToday() {
            this.date = new Date();
        },

        handleShortcutClick(shortcut) {
            if (shortcut.onClick) {
                shortcut.onClick(this);
            }
        },

        resetView() {
            this.minTimePickerVisible = false;
            this.maxTimePickerVisible = false;
        },

        setTime(date, value) {
            let oldDate = new Date(date.getTime());
            let hour = value.getHours();
            let minute = value.getMinutes();
            let second = value.getSeconds();
            oldDate.setHours(hour);
            oldDate.setMinutes(minute);
            oldDate.setSeconds(second);
            return new Date(oldDate.getTime());
        },

        handleMinTimePick(value, visible, first) {
            this.minDate = this.minDate || new Date();
            if (value) {
                this.minDate = this.setTime(this.minDate, value);
            }

            if (!first) {
                this.minTimePickerVisible = visible;
            }
        },

        handleMaxTimePick(value, visible, first) {
            if (!this.maxDate) {
                const now = new Date();
                if (now >= this.minDate) {
                    this.maxDate = new Date();
                }
            }

            if (this.maxDate && value) {
                this.maxDate = this.setTime(this.maxDate, value);
            }

            if (!first) {
                this.maxTimePickerVisible = visible;
            }
        },

        prevMonth() {
            this.date = prevMonth(this.date);
        },

        nextMonth() {
            this.date = nextMonth(this.date);
        },

        nextYear() {
            const date = this.date;
            date.setFullYear(date.getFullYear() + 1);
            this.resetDate();
        },

        prevYear() {
            const date = this.date;
            date.setFullYear(date.getFullYear() - 1);
            this.resetDate();
        },

        handleConfirm(visible = false) {
            this.$emit('pick', [this.minDate, this.maxDate], visible);
        },

        resetDate() {
            this.date = new Date(this.date);
        }
    },

    components: { TimePicker, DateTable, XInput }
};

</script>
