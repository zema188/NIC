<template>
    <div class="form container">
        <div class="form__fields">
            <div class="form__fields-item"
                v-for="(field, index) in fields"
                :key="index"
            >
                <p>{{ field.name }}</p>
                <input
                    v-model="field.value"
                    :placeholder="field.name"
                    @input="handlerFieldInput(field.name, field.value)"
                    @keypress="validateInput($event)"
                />
            </div>
            <button class="form__fields-item form__submit"
                @click="handleSubmit()"
            >
                Submit
            </button>
        </div>
        <div class="form__labels">
            <label v-for="(field, index) in fields"
                :key="index"
            >
                {{ field.value }}
            </label>
            <label>Текущее состояние localStorage: {{ localStorageState }}</label>
        </div>
        <div class="form__info">
            <div class="form__info-event"
                v-for="(event, index) in events"
                :key="index"
            >
                {{ event }}
            </div>
        </div>
    </div>
</template>

<script setup>
import { onMounted, reactive, ref, watch } from 'vue';

const fields = reactive([
    { name: 'price', value: '' },
    { name: 'qty', value: '' },
    { name: 'amount', value: '' },
]);

const counter = ref(localStorage.getItem('counter') ? parseInt(localStorage.getItem('counter')) : 0);
const events = reactive([]);
const localStorageState = ref('');

const updateLocalStorageState = () => {
    localStorageState.value = JSON.stringify({
        counter: localStorage.getItem('counter'),
        price: localStorage.getItem('price'),
        qty: localStorage.getItem('qty'),
        amount: localStorage.getItem('amount'),
    });
};

const debounce = (func, delay) => {
    console.log("test1")
    let timeout;
    return (...args) => {
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(this, args), delay);
    };
};

const handlerFieldInput = debounce((field, value) => {
    events.unshift(`Изменено поле ${field} на ${value}`);
    
    const priceField = fields.find(f => f.name === 'price');
    const qtyField = fields.find(f => f.name === 'qty');
    const amountField = fields.find(f => f.name === 'amount');

    if (field === 'price' || field === 'qty') {
        const computedAmount = Math.floor(parseFloat(priceField.value) * parseFloat(qtyField.value));
        if (!isNaN(computedAmount)) {
            amountField.value = computedAmount.toString();
            events.unshift(`Изменено поле amount на ${amountField.value}`);
        }
    } else if (field === 'amount') {
        if (parseFloat(priceField.value) !== 0 && parseFloat(qtyField.value) !== 0) {
            priceField.value = (parseFloat(amountField.value) / parseFloat(qtyField.value)).toFixed(2);
            events.unshift(`Изменено поле price на ${priceField.value}`);
        }
    }
}, 300);

const validateInput = (event) => {
    const char = String.fromCharCode(event.keyCode);
    if (!/^\d+$/.test(char)) {
        event.preventDefault();
    }
};

const sendDataToBackend = (dataToSave) => {
    return new Promise((resolve) => {
        setTimeout(() => {
            const success = parseInt(dataToSave.amount) % 2 === 0;
            resolve({ success });
        }, 1000);
    });
};

const handleSubmit = async () => {
    const dataToSave = {
        counter: counter.value++,
        price: fields.find(f => f.name === 'price').value,
        qty: fields.find(f => f.name === 'qty').value,
        amount: fields.find(f => f.name === 'amount').value,
    };

    events.unshift(`Отправка данных: ${JSON.stringify(dataToSave)}; Текущее состояние localStorage: ${localStorageState.value}`);
    
    const response = await sendDataToBackend(dataToSave);
    events.unshift(`Ответ от сервера: ${JSON.stringify(response)}; Отправленные данные: ${JSON.stringify(dataToSave)}; Текущее состояние localStorage: ${localStorageState.value}`);
    
    if (response.success) {
        localStorage.setItem('counter', counter.value);
        localStorage.setItem('price', dataToSave.price);
        localStorage.setItem('qty', dataToSave.qty);
        localStorage.setItem('amount', dataToSave.amount);
        updateLocalStorageState();
    } else {
        counter.value--;
    }
};

watch([fields, counter], () => {
    updateLocalStorageState();
}, { deep: true });

onMounted(() => {
    updateLocalStorageState();
});
</script>

<style lang="scss" scoped>
input {
    border: 1px solid #969696;
    padding: 10px 5px;
}
.form {
    padding-top: 20px;
    padding-bottom: 20px;

    &__fields {
        display: flex;
        gap: 10px;
        margin-bottom: 20px;
        align-items: flex-end;
    }

    &__labels {
        display: flex;
        gap: 10px;
        margin-bottom: 20px;
        & label {
            flex: 0 0 calc(25% - 7.5px);
            word-break: break-all;
        }
    }

    &__info {
    }

    &__info-event {
        margin-bottom: 30px;
    }

    &__fields-item {
        flex: 0 0 calc(25% - 7.5px);
    }

    &__submit {
        display: flex;
        justify-content: center;
        align-items: center;
        text-align: center;
        background: blue;
        color: #fff;
        border-radius: 8px;
        padding: 12px 15px;
        cursor: pointer;
    }
}
.container {
}
</style>
