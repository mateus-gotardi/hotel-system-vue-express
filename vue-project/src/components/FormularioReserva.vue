<template>
    <div class="modal-container">
        <div class="modal">
            <div class="modal-header">
                <h2>{{ reservaStore.editMode ? 'Editar Reserva' : 'Criar Reserva' }}</h2>
                <button @click="() => {
                    reservaStore.fecharModal()
                    reservaStore.removerReserva()
                    reservaStore.fetchReservas(hotelStore.hotel.id)
                    reservaStore.sairModoEdicao()
                }">X</button>
            </div>
            <div class="modal-form" v-if="reservaStore.reserva.id !== -1 || !reservaStore.editMode">
                <div class="select-hotel" v-if="!reservaStore.editMode">
                    <label for="selectHotel">Selecione um hotel: </label>
                    <select id="selectHotel" v-model="hotelStore.hotel">
                        <option v-for="hotel in hotelStore.hoteis" :key="hotel.id" :value="hotel">{{ hotel.nome }}
                        </option>
                    </select>
                </div>
                <p v-if="hotelStore.hotel.nome != ''">Hotel: {{ hotelStore.hotel.nome }}</p>
                <div class="inputContainer">
                    <label for="numeroreserva">Reserva*</label>
                    <input v-if="!reservaStore.editMode" required type="text" placeholder="ex: 123"
                        v-model="reservaStore.reserva.numeroreserva" />
                    <p v-if="reservaStore.editMode">{{ reservaStore.reserva.numeroreserva }}</p>
                </div>
                <div class="inputContainer">
                    <label for="apartamento">Apartamento*</label>
                    <input required type="text" placeholder="ex: 50" v-model="reservaStore.reserva.apartamento" />
                </div>
                <div class="inputContainer">
                    <label for="checkin">Data check in*</label>
                    <input required type="date" placeholder="2022-10-01" v-model="reservaStore.reserva.datacheckin" />
                </div>
                <div class="inputContainer">
                    <label for="checkout">Data check out</label>
                    <input required type="date" placeholder="2022-10-01" v-model="reservaStore.reserva.datacheckout" />
                </div>
                <div class="inputContainer">
                    <label for="status">Status*</label>
                    <input required type="text" placeholder="ex: 1" v-model="reservaStore.reserva.status" />
                </div>
                <div class="hospedes-container">
                    <p>Hospedes*</p>
                    <div class="hospede" v-for="(hospede, index) in reservaStore.reserva.hospedes" :key="index">
                        <input type="text" placeholder="Nome" v-model="hospede.nome">
                        <input type="text" placeholder="Sobrenome" v-model="hospede.sobrenome">
                        <button @click="removerHospede(index)">-</button>
                    </div>
                    <button @click="criarHospede">+</button>
                </div>
                <p class="error" v-if="error !== ''">{{ error }}</p>
                <div class="buttonContainer">
                    <button @click="registerReserva">{{ reservaStore.editMode ? 'Editar Reserva' : 'Criar Reserva'
                    }}</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import api from '../api'
import { useHotelStore } from '@/stores/hotel';
import { useReservaStore } from '@/stores/reserva';

const hotelStore = useHotelStore()
const reservaStore = useReservaStore()

let error = ref('')

function criarHospede() {
    reservaStore.adicionarHospede()
}
function removerHospede(index: number) {
    reservaStore.removerHospede(index)
}
async function registerReserva() {
    if (reservaStore.reserva.apartamento == '' ||
        reservaStore.reserva.datacheckin == '' ||
        reservaStore.reserva.status == '' ||
        reservaStore.reserva.numeroreserva == -1 ||
        (reservaStore.reserva.datacheckout && reservaStore.reserva.datacheckin > reservaStore.reserva.datacheckout) ||
        reservaStore.reserva.hospedes?.length == 0) {
        if (reservaStore.reserva.numeroreserva == -1) {
            error.value = 'Numero da reserva não pode ser negativo';
            return
        }
        else if (reservaStore.reserva.datacheckout && reservaStore.reserva.datacheckin > reservaStore.reserva.datacheckout) {
            error.value = 'Data check in não pode ser maior que data checkout';
        }
        else if (reservaStore.reserva.hospedes?.length == 0) {
            error.value = 'Reserva precisa ter pelo menos um hospede'
        } else {
            error.value = 'Todos os campos com * são obrigatórios';
        }
        return
    } else {
        error.value = ''
        const temNomeOuSobrenomeVazio = reservaStore.reserva.hospedes?.some(pessoa => pessoa.nome === '' || pessoa.sobrenome === '');
        if (temNomeOuSobrenomeVazio) {
            error.value = 'Preencha os nomes dos hospedes'
            return
        }
    }
    try {
        const reserva = reservaStore.editMode ? await api.reservas.update({
            apartamento: reservaStore.reserva.apartamento,
            idhotel: hotelStore.hotel.id,
            numeroreserva: reservaStore.reserva.numeroreserva,
            datacheckin: reservaStore.reserva.datacheckin,
            datacheckout: reservaStore.reserva.datacheckout,
            hospedes: reservaStore.reserva.hospedes,
            status: reservaStore.reserva.status
        }) : await api.reservas.create({
            apartamento: reservaStore.reserva.apartamento,
            idhotel: hotelStore.hotel.id,
            numeroreserva: reservaStore.reserva.numeroreserva,
            datacheckin: reservaStore.reserva.datacheckin,
            datacheckout: reservaStore.reserva.datacheckout,
            hospedes: reservaStore.reserva.hospedes,
            status: reservaStore.reserva.status
        })
        if (reserva.data == 'reservado com sucesso') {
            reservaStore.fecharModal()
            reservaStore.removerReserva()
            reservaStore.sairModoEdicao()
            reservaStore.fetchReservas(hotelStore.hotel.id)
        }
    } catch (e: any) {
        console.log('erro')
        console.log(e.response.data.message)
        error.value = e.response.data.message
    }
}

</script>

<style>
.error {
    color: red;
    padding: 0;
    margin: 0;
}

.modal-form {
    width: 100%
}

.modal-container {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: rgba(0, 0, 0, 0.7);
}

@media (min-width: 1024px) {
    .modal {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 1rem;
        top: 5rem;
        right: 5rem;
        background-color: var(--color-background-soft);
        padding: 1rem 2rem;
        border-radius: 1rem;
        width: 30%;
    }

    .inputContainer {
        display: flex;
        flex-direction: column;
        gap: 0.2rem;
        width: 100%;
    }

    .buttonContainer {
        display: flex;
        flex-direction: column;
        gap: 0.2rem;
        padding-top: 1rem;
    }

    .modal-header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        width: 100%;
    }
}
</style>
  