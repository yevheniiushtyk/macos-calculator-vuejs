<template>
    <div class="calculator">
        <div class="top-bar">
            <div class="dot-container">
                <div class="dot red">
                    <div class="icon">×</div>
                </div>
                <div class="dot yellow">
                    <div class="icon">−</div>
                </div>
                <div class="dot green">
                    <div class="icon">+</div>
                </div>
            </div>
        </div>
        <div class="display">
            <div class="result">{{ displayValue }}</div>
        </div>
        <div class="buttons">
            <button v-for="button in buttons" :key="button.label" :class="[button.class, { active: isActive(button) }]"
                :data-operator="button.operator" @click="handleClick(button)">
                {{ button.label }}
            </button>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                displayValue: '0',
                firstOperand: null,
                operator: null,
                waitingForSecondOperand: false,
                activeOperator: null,
                buttons: [
                    { label: 'AC', class: 'button function', id: 'clear' },
                    { label: '+/-', class: 'button function' },
                    { label: '%', class: 'button function' },
                    { label: '÷', class: 'button operator', operator: '÷' },
                    { label: '7', class: 'button number' },
                    { label: '8', class: 'button number' },
                    { label: '9', class: 'button number' },
                    { label: '×', class: 'button operator', operator: '×' },
                    { label: '4', class: 'button number' },
                    { label: '5', class: 'button number' },
                    { label: '6', class: 'button number' },
                    { label: '-', class: 'button operator', operator: '-' },
                    { label: '1', class: 'button number' },
                    { label: '2', class: 'button number' },
                    { label: '3', class: 'button number' },
                    { label: '+', class: 'button operator', operator: '+' },
                    { label: '0', class: 'button number zero' },
                    { label: ',', class: 'button number' },
                    { label: '=', class: 'button operator', operator: '=' },
                ],
            };
        },
        methods: {
            handleClick(button) {
                if (button.class.includes('number') && button.label !== ',') {
                    this.inputDigit(button.label);
                } else if (button.class.includes('operator')) {
                    this.handleOperator(button.operator);
                } else if (button.label === ',') {
                    this.inputDecimal();
                } else {
                    this.handleFunction(button.label);
                }
                this.updateDisplay();
            },
            updateDisplay() {
                const display = document.querySelector('.result');

                display.textContent = this.displayValue;

                const displayContainer = document.querySelector('.display');
                const maxWidth = displayContainer.clientWidth - 40;
                let fontSize = 80;

                display.style.fontSize = `${fontSize}px`;

                while (display.scrollWidth > maxWidth) {
                    fontSize -= 1;
                    display.style.fontSize = `${fontSize}px`;
                }

                const clearButton = this.buttons.find(button => button.id === 'clear');
                clearButton.label = this.displayValue === '0' && this.firstOperand === null ? 'AC' : 'C';
            },
            inputDigit(digit) {
                if (this.waitingForSecondOperand) {
                    this.displayValue = digit;
                    this.waitingForSecondOperand = false;
                } else {
                    this.displayValue = this.displayValue === '0' ? digit : this.displayValue + digit;
                }
            },
            inputDecimal() {
                if (this.waitingForSecondOperand) {
                    this.displayValue = '0,';
                    this.waitingForSecondOperand = false;
                    return;
                }
                if (!this.displayValue.includes(',')) {
                    this.displayValue += ',';
                }
            },
            handleOperator(nextOperator) {
                const inputValue = parseFloat(this.displayValue.replace(',', '.'));

                if (this.operator && this.waitingForSecondOperand) {
                    this.operator = nextOperator;
                    this.updateActiveOperator(nextOperator);
                    return;
                }

                if (this.firstOperand === null && !isNaN(inputValue)) {
                    this.firstOperand = inputValue;
                } else if (this.operator) {
                    const result = this.calculate(this.firstOperand, inputValue, this.operator);
                    this.displayValue = `${parseFloat(result.toFixed(7))}`.replace('.', ',');
                    this.firstOperand = result;
                }

                this.waitingForSecondOperand = true;
                this.operator = nextOperator;
                this.updateActiveOperator(nextOperator);
                this.updateDisplay();
            },
            updateActiveOperator(nextOperator) {
                if (nextOperator === '=') {
                    this.activeOperator = '=';
                    setTimeout(() => {
                        this.activeOperator = null;
                    }, 150);
                } else {
                    this.activeOperator = nextOperator;
                }
            },
            isActive(button) {
                return button.operator === this.activeOperator;
            },
            calculate(first, second, op) {
                switch (op) {
                    case '+':
                        return first + second;
                    case '-':
                        return first - second;
                    case '×':
                        return first * second;
                    case '÷':
                        return second === 0 ? 'Error' : first / second;
                    default:
                        return second;
                }
            },
            resetCalculator() {
                this.displayValue = '0';
                this.firstOperand = null;
                this.operator = null;
                this.waitingForSecondOperand = false;
                this.updateActiveOperator(null);
                this.updateDisplay();
            },
            handleFunction(func) {
                if (func === 'AC') {
                    this.resetCalculator();
                } else if (func === 'C') {
                    this.displayValue = '0';
                    this.updateDisplay();
                } else if (func === '+/-') {
                    this.displayValue = (parseFloat(this.displayValue.replace(',', '.')) * -1).toString().replace('.', ',');
                    this.updateDisplay();
                } else if (func === '%') {
                    this.displayValue = (parseFloat(this.displayValue.replace(',', '.')) / 100).toString().replace('.', ',');
                    this.updateDisplay();
                }
            },
            handleKeydown(event) {
                const { key, altKey } = event;
                let button = null;

                if (key >= '0' && key <= '9') {
                    button = this.buttons.find(b => b.label === key);
                }
                else if (key === '.' || key === ',') {
                    button = this.buttons.find(b => b.label === ',');
                }
                else if (key === '+') {
                    button = this.buttons.find(b => b.label === '+');
                }
                else if (key === '-') {
                    if (altKey) {
                        button = this.buttons.find(b => b.id === 'plus-minus');
                    } else {
                        button = this.buttons.find(b => b.label === '-');
                    }
                }
                else if (key === '*') {
                    button = this.buttons.find(b => b.label === '×');
                }
                else if (key === '/') {
                    button = this.buttons.find(b => b.label === '÷');
                }
                else if (key === 'Enter' || key === '=') {
                    button = this.buttons.find(b => b.label === '=');
                }
                else if (key === 'Backspace') {
                    if (this.displayValue.length > 1) {
                        this.displayValue = this.displayValue.slice(0, -1);
                    } else {
                        this.displayValue = '0';
                    }
                    this.updateDisplay();
                    return;
                }
                else if (key === 'Escape') {
                    button = this.buttons.find(b => b.id === 'clear');
                    if (button) {
                        this.handleFunction(altKey ? 'AC' : (button.textContent || 'C'));
                    }
                }
                else if (key.toUpperCase() === 'C') {
                    button = this.buttons.find(b => b.id === 'clear');
                    if (button) {
                        this.handleFunction('C');
                    }
                }
                else if (key === '%') {
                    button = this.buttons.find(b => b.label === '%');
                }

                if (button) {
                    this.handleClick(button);
                    this.simulateButtonClick(button);
                }
            },
            simulateButtonClick(button) {
                if (button && button.label) {
                    const buttonElement = Array.from(this.$el.querySelectorAll('button')).find(el => el.textContent.trim() === button.label);
                    if (buttonElement) {
                        buttonElement.classList.add('active');
                        setTimeout(() => buttonElement.classList.remove('active'), 150);
                    }
                }
            }
        },
        mounted() {
            document.title = 'Calculator';

            this.updateDisplay();
            document.addEventListener('keydown', this.handleKeydown);
        },
        beforeUnmount() {
            document.removeEventListener('keydown', this.handleKeydown);
        }
    };
</script>

<style scoped>
    .calculator {
        width: 320px;
        border-radius: 15px;
        overflow: hidden;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        position: relative;
        background-color: #242423;
        border: 1px solid rgb(75, 75, 74);
    }

    .top-bar {
        display: flex;
        justify-content: flex-start;
        padding: 10px;
        background-color: #242423;
        position: relative;
    }

    .dot {
        width: 12px;
        height: 12px;
        border-radius: 50%;
        margin-right: 8px;
        position: relative;
        z-index: 1;
        display: flex;
        align-items: center;
        justify-content: center;
        color: #000;
        user-select: none;
    }

    .dot .icon {
        display: none;
        font-size: 10px;
        font-weight: bold;
    }

    .dot-container {
        display: flex;
        position: relative;
    }

    .dot-container::before {
        content: "";
        position: absolute;
        top: -5px;
        left: -5px;
        right: -5px;
        bottom: -5px;
        z-index: 0;
    }

    .dot-container:hover .icon {
        display: block;
    }

    .red {
        background-color: rgb(237, 105, 94);
    }

    .red .icon {
        color: rgb(151, 9, 9);
    }

    .yellow {
        background-color: rgb(245, 189, 79);
    }

    .yellow .icon {
        color: rgb(170, 102, 15);
    }

    .green {
        background-color: rgb(97, 196, 84);
    }

    .green .icon {
        color: rgb(4, 93, 5);
    }

    .display {
        background-color: #242423;
        color: #fff;
        text-align: right;
        padding: 0px 20px 5px 20px;
        font-size: 80px;
        display: flex;
        justify-content: flex-end;
        align-items: flex-end;
        min-height: 80px;
        overflow: hidden;
        white-space: nowrap;
    }

    .result {
        width: 100%;
        word-wrap: break-word;
    }

    .buttons {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        grid-gap: 1px;
    }

    .button {
        padding: 20px;
        border: none;
        background-color: #5b5a59;
        font-size: 1.7em;
        color: #fff;
        cursor: pointer;
        user-select: none;
        transition: background-color 0.1s;
        outline: none;
    }

    .button.function {
        background-color: rgb(57, 57, 57);
    }

    .button.operator {
        background-color: rgb(242, 162, 59);
        color: #fff;
        position: relative;
    }

    .button.operator.active {
        background-color: rgb(242, 162, 59);
        color: #fff;
    }

    .button.operator.active::after {
        content: '';
        position: absolute;
        top: -1px;
        left: -1px;
        right: -1px;
        bottom: -1px;
        border: 3px solid rgb(36, 36, 35);
        border-radius: 2px;
        pointer-events: none;
    }

    .button.operator[data-operator="="].active::after {
        content: none;
    }

    .button.zero {
        grid-column: span 2;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 20px;
    }

    .buttons .button:active,
    .buttons .button.active {
        transition: none;
        background-color: rgb(156, 156, 155);
    }

    .buttons .button.function:active,
    .buttons .button.function.active {
        background-color: rgb(90, 90, 89);
    }

    .buttons .button.operator:active,
    .buttons .button.operator.active {
        background-color: rgb(193, 128, 45);
    }

    @media (max-width: 360px) {
        .calculator {
            width: 100%;
            border-radius: 0;
        }
    }
</style>
