<template>
    <div class="datepicker control" :class="[size, {'is-expanded': expanded}]">
        <b-dropdown v-if="!isMobile || inline"
            ref="dropdown"
            :inline="inline">
            <b-input v-if="!inline"
                ref="input"
                slot="trigger"
                autocomplete="off"
                :value="formattedDateSelected"
                :placeholder="placeholder"
                :size="size"
                :icon="icon"
                :icon-pack="iconPack"
                :loading="loading"
                v-bind="$attrs"
                @change.native="(event) => onChange(event.target.value)"
                @focus="$emit('focus', $event)"
                @blur="$emit('blur', $event) && checkHtml5Validity()">
            </b-input>

            <b-dropdown-item custom>
                <header class="datepicker-header">
                    <div class="pagination field is-centered">
                        <a v-if="!isFirstMonth"
                            class="pagination-previous"
                            role="button"
                            tabindex="0"
                            aria-label="Decrement Month"
                            @click="decrementMonth"
                            @keydown.enter="decrementMonth"
                            @keydown.space.prevent="decrementMonth">

                            <b-icon icon="chevron_left"
                                both
                                type="is-primary is-clickable">
                            </b-icon>
                        </a>
                        <a v-show="!isLastMonth"
                            class="pagination-next"
                            role="button"
                            tabindex="0"
                            aria-label="Increment Month"
                            @click="incrementMonth"
                            @keydown.enter="incrementMonth"
                            @keydown.space.prevent="incrementMonth">

                            <b-icon icon="chevron_right"
                                both
                                type="is-primary is-clickable">
                            </b-icon>
                        </a>
                        <div class="pagination-list">
                            <b-field>
                                <b-select v-model="focusedDateData.month" aria-label="Select Month">
                                    <option v-for="(month, index) in Object.values(monthNames)"
                                        :value="index"
                                        :key="month">
                                        {{month}}
                                    </option>
                                </b-select>
                                <b-select v-model="focusedDateData.year"
                                    aria-label="Select Year">
                                    <option v-for="year in listOfYears"
                                        :value="year"
                                        :key="year">
                                        {{year}}
                                    </option>
                                </b-select>
                            </b-field>
                        </div>
                    </div>
                </header>

                <b-datepicker-table v-model="dateSelected"
                    :day-names="dayNames"
                    :month-names="monthNames"
                    :first-day-of-week="firstDayOfWeek"
                    :min-date="minDate"
                    :max-date="maxDate"
                    :focused="focusedDateData"
                    @close="$refs.dropdown.isActive = false">
                </b-datepicker-table>

                <footer v-if="$slots.default !== undefined && $slots.default.length"
                    class="datepicker-footer">
                    <slot></slot>
                </footer>
            </b-dropdown-item>
        </b-dropdown>

        <b-input v-else
            ref="input"
            type="date"
            autocomplete="off"
            :value="formatYYYYMMDD(value)"
            :placeholder="placeholder"
            :size="size"
            :icon="icon"
            :icon-pack="iconPack"
            :loading="loading"
            :max="formatYYYYMMDD(maxDate)"
            :min="formatYYYYMMDD(minDate)"
            v-bind="$attrs"
            @change.native="onChangeNativePicker"
            @focus="$emit('focus', $event)"
            @blur="$emit('blur', $event) && checkHtml5Validity()">
        </b-input>
    </div>
</template>

<script>
    import FormElementMixin from '../../utils/FormElementMixin'
    import { isMobile } from '../../utils/helpers'
    import config from '../../utils/config'

    import { Dropdown, DropdownItem } from '../dropdown'
    import Input from '../input'
    import Field from '../field'
    import Select from '../select'
    import Icon from '../icon'
    import DatepickerTable from './DatepickerTable'

    export default {
        name: 'bDatepicker',
        inheritAttrs: false,
        mixins: [FormElementMixin],
        components: {
            [DatepickerTable.name]: DatepickerTable,
            [Input.name]: Input,
            [Field.name]: Field,
            [Select.name]: Select,
            [Icon.name]: Icon,
            [Dropdown.name]: Dropdown,
            [DropdownItem.name]: DropdownItem
        },
        props: {
            value: Date,
            dayNames: {
                type: Array,
                default() {
                    return [
                        'Su',
                        'M',
                        'Tu',
                        'W',
                        'Th',
                        'F',
                        'S'
                    ]
                }
            },
            monthNames: {
                type: Array,
                default() {
                    return [
                        'January',
                        'February',
                        'March',
                        'April',
                        'May',
                        'June',
                        'July',
                        'August',
                        'September',
                        'October',
                        'November',
                        'December'
                    ]
                }
            },
            firstDayOfWeek: {
                type: Number,
                default: 0
            },
            inline: Boolean,
            minDate: Date,
            maxDate: Date,
            focusedDate: Date,
            placeholder: String,
            dateFormatter: {
                type: Function,
                default: (date) => {
                    if (typeof config.defaultDateFormatter === 'function') {
                        return config.defaultDateFormatter(date)
                    } else {
                        return date.toLocaleDateString()
                    }
                }
            },
            dateParser: {
                type: Function,
                default: (date) => {
                    if (typeof config.defaultDateParser === 'function') {
                        return config.defaultDateParser(date)
                    } else {
                        return new Date(Date.parse(date))
                    }
                }
            }
        },
        data() {
            const focusedDate = this.value || this.focusedDate || new Date()

            return {
                dateSelected: this.value,
                focusedDateData: {
                    month: focusedDate.getMonth(),
                    year: focusedDate.getFullYear()
                },
                _elementRef: 'input',
                _isDatepicker: true
            }
        },
        computed: {
            /*
            * Returns an array of years for the year dropdown. If earliest/latest
            * dates are set by props, range of years will fall within those dates.
            */
            listOfYears() {
                const latestYear = this.maxDate
                ? this.maxDate.getFullYear() : new Date().getFullYear() + 3

                const earliestYear = this.minDate
                ? this.minDate.getFullYear() : 1900

                const arrayOfYears = []
                for (let i = earliestYear; i <= latestYear; i++) {
                    arrayOfYears.push(i)
                }

                return arrayOfYears.reverse()
            },

            formattedDateSelected() {
                return this.formatValue(this.dateSelected)
            },

            isFirstMonth() {
                if (!this.minDate) return false
                const dateToCheck = new Date(this.focusedDateData.year, this.focusedDateData.month)
                const date = new Date(this.minDate.getFullYear(), this.minDate.getMonth())
                return (dateToCheck <= date)
            },

            isLastMonth() {
                if (!this.maxDate) return false
                const dateToCheck = new Date(this.focusedDateData.year, this.focusedDateData.month)
                const date = new Date(this.maxDate.getFullYear(), this.maxDate.getMonth())
                return (dateToCheck >= date)
            },

            isMobile() {
                return isMobile.any()
            }
        },
        watch: {
            /*
            * Emit input event with selected date as payload, set isActive to false.
            * Update internal focusedDateData
            */
            dateSelected(value) {
                const currentDate = !value ? new Date() : value
                this.focusedDateData = {
                    month: currentDate.getMonth(),
                    year: currentDate.getFullYear()
                }
                this.$emit('input', value)
                if (this.$refs.dropdown) {
                    this.$refs.dropdown.isActive = false
                }
            },

            /**
             * When v-model is changed:
             *   1. Update internal value.
             *   2. If it's invalid, validate again.
             */
            value(value) {
                this.dateSelected = value

                !this.isValid && this.$refs.input.checkHtml5Validity()
            }
        },
        methods: {
            /*
            * Emit input event with selected date as payload for v-model in parent
            */
            updateSelectedDate(date) {
                this.dateSelected = date
            },

            /*
            * Parse string into date
            */
            onChange(value) {
                const date = this.dateParser(value)
                if (date && !isNaN(date)) {
                    this.dateSelected = date
                } else {
                    this.dateSelected = null
                    // force refresh when not valid date
                    this.$nextTick(() => {
                        // see computed 'formattedDateSelected'
                        this.$refs.input.$refs.input.value = this.formatValue(this.dateSelected)
                    })
                }
            },

            /*
            * Format date into string
            */
            formatValue(value) {
                if (value && !isNaN(value)) {
                    return this.dateFormatter(value)
                } else {
                    return null
                }
            },

            /*
            * Either decrement month by 1 if not January or decrement year by 1
            * and set month to 11 (December)
            */
            decrementMonth() {
                if (this.focusedDateData.month > 0) {
                    this.focusedDateData.month -= 1
                } else {
                    this.focusedDateData.month = 11
                    this.focusedDateData.year -= 1
                }
            },

            /*
            * Either increment month by 1 if not December or increment year by 1
            * and set month to 0 (January)
            */
            incrementMonth() {
                if (this.focusedDateData.month < 11) {
                    this.focusedDateData.month += 1
                } else {
                    this.focusedDateData.month = 0
                    this.focusedDateData.year += 1
                }
            },

            /*
            * Return name of month full-length
            */
            nameOfMonth(month) {
                const months = {}
                for (let i = 0; i < this.monthNames.length; i++) {
                    months[i] = this.monthNames[i]
                }

                return months[month]
            },

            /*
            * Format date into string 'YYYY-MM-DD'
            */
            formatYYYYMMDD(value) {
                if (value && !isNaN(new Date(value))) {
                    const date = new Date(value)
                    const year = date.getFullYear()
                    const month = date.getMonth() + 1
                    const day = date.getDate()
                    return year + '-' + ((month < 10 ? '0' : '') + month) + '-' + ((day < 10 ? '0' : '') + day)
                }
                return ''
            },

            /*
            * Format date into string 'YYYY-MM-DD'
            */
            onChangeNativePicker(event) {
                // YYYY-MM-DD
                const date = event.target.value
                this.dateSelected = date ? new Date(date) : null
            }
        }
    }
</script>
