<template>
  <div v-if="open" class="checkout-overlay" @click.self="handleClose">
    <div class="checkout-modal" role="dialog" aria-modal="true" aria-label="Fechar pedido">
      <header class="checkout-header">
        <div class="steps">
          <div v-for="step in steps" :key="step.id" class="step" :class="{ active: currentStep === step.id, done: currentStep > step.id }">
            <span class="step-index">{{ step.id }}</span>
            <span class="step-label">{{ step.label }}</span>
          </div>
        </div>

        <button class="close-button" type="button" aria-label="Fechar" @click="handleClose">✕</button>
      </header>

      <section class="checkout-body">
        <!-- Etapa 1: Entrega -->
        <div v-if="currentStep === 1" class="checkout-step">
          <h3>Como você quer receber?</h3>

          <div class="option-list">
            <label class="option-card" :class="{ selected: deliveryMethod === 'retirada' }">
              <input type="radio" value="retirada" v-model="deliveryMethod" />
              <div>
                <strong>Retirada no local</strong>
                <p>Sem custo de entrega</p>
              </div>
              <span class="option-price">Grátis</span>
            </label>

            <label class="option-card" :class="{ selected: deliveryMethod === 'entrega' }">
              <input type="radio" value="entrega" v-model="deliveryMethod" />
              <div>
                <strong>Entrega</strong>
                <p>Receba no seu endereço</p>
              </div>
              <span class="option-price">{{ currency(deliveryFee) }}</span>
            </label>
          </div>

          <label v-if="deliveryMethod === 'entrega'" class="field">
            <span>Endereço de entrega</span>
            <input type="text" v-model="address" placeholder="Rua, número, bairro" />
          </label>
        </div>

        <!-- Etapa 2: Pagamento -->
        <div v-if="currentStep === 2" class="checkout-step">
          <h3>Forma de pagamento</h3>

          <div class="option-list">
            <label class="option-card" :class="{ selected: paymentMethod === 'dinheiro' }">
              <input type="radio" value="dinheiro" v-model="paymentMethod" />
              <div><strong>Dinheiro</strong></div>
            </label>

            <label class="option-card" :class="{ selected: paymentMethod === 'cartao_credito' }">
              <input type="radio" value="cartao_credito" v-model="paymentMethod" />
              <div><strong>Cartão de crédito</strong></div>
            </label>

            <label class="option-card" :class="{ selected: paymentMethod === 'cartao_debito' }">
              <input type="radio" value="cartao_debito" v-model="paymentMethod" />
              <div><strong>Cartão de débito</strong></div>
            </label>

            <label class="option-card" :class="{ selected: paymentMethod === 'pix' }">
              <input type="radio" value="pix" v-model="paymentMethod" />
              <div><strong>Pix</strong></div>
            </label>
          </div>

          <div v-if="paymentMethod === 'dinheiro'" class="field">
            <span>Vai precisar de troco?</span>

            <div class="toggle-row">
              <button type="button" :class="{ selected: needsChange === true }" @click="needsChange = true">Sim</button>
              <button type="button" :class="{ selected: needsChange === false }" @click="needsChange = false; changeFor = ''">Não</button>
            </div>
          </div>

          <label v-if="paymentMethod === 'dinheiro' && needsChange" class="field">
            <span>Troco para quanto?</span>
            <input type="text" v-model="changeFor" placeholder="Ex: R$ 100,00" />
          </label>
        </div>

        <!-- Etapa 3: Resumo -->
        <div v-if="currentStep === 3" class="checkout-step">
          <h3>Quase lá!</h3>
          <p class="summary-subtitle">Confira os detalhes do seu pedido antes de confirmar.</p>

          <ul class="summary-items">
            <li v-for="item in items" :key="item.product.id">
              <span>{{ item.quantity }}x {{ item.product.name }}</span>
              <strong>{{ currency(item.product.price * item.quantity) }}</strong>
            </li>
          </ul>

          <div class="summary-row"><span>Subtotal</span><strong>{{ currency(subtotal) }}</strong></div>
          <div class="summary-row"><span>Desconto</span><strong>{{ discount > 0 ? `- ${currency(discount)}` : currency(0) }}</strong></div>
          <div class="summary-row"><span>Taxa de entrega</span><strong>{{ deliveryMethod === 'entrega' ? currency(deliveryFee) : 'Grátis' }}</strong></div>
          <div class="summary-row total"><span>Total</span><strong>{{ currency(total) }}</strong></div>

          <div class="summary-row"><span>Entrega</span><strong>{{ deliveryMethod === 'entrega' ? 'Entrega' : 'Retirada no local' }}</strong></div>
          <div class="summary-row"><span>Tempo estimado</span><strong>{{ estimatedTime }}</strong></div>
          <div v-if="deliveryMethod === 'entrega'" class="summary-row"><span>Endereço</span><strong>{{ address }}</strong></div>
          <div class="summary-row"><span>Pagamento</span><strong>{{ paymentLabel }}</strong></div>
          <div v-if="paymentMethod === 'dinheiro' && needsChange" class="summary-row"><span>Troco para</span><strong>{{ changeFor }}</strong></div>
        </div>
      </section>

      <footer class="checkout-footer">
        <button v-if="currentStep > 1" type="button" class="button secondary" @click="currentStep -= 1">Voltar</button>

        <button v-if="currentStep < 3" type="button" class="button primary" :disabled="!canAdvance" @click="currentStep += 1">Continuar</button>

        <button v-else type="button" class="button primary" @click="confirmOrder">Confirmar pedido</button>
      </footer>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'

type Product = {
  id: number
  name: string
  description: string
  price: number
  image: string
  category: string
}

type CartItem = { product: Product; quantity: number }

type Order = {
  items: CartItem[]
  deliveryMethod: 'retirada' | 'entrega'
  address: string
  deliveryFee: number
  paymentMethod: 'dinheiro' | 'cartao_credito' | 'cartao_debito' | 'pix'
  needsChange: boolean
  changeFor: string
  subtotal: number
  discount: number
  total: number
  estimatedTime: string
}

const props = defineProps<{ open: boolean; items: CartItem[]; subtotal: number; discount: number }>()
const emit = defineEmits<{ close: []; confirm: [order: Order] }>()

const deliveryFee = 7
const estimatedTime = '30-40 min'

const steps = [
  { id: 1, label: 'Entrega' },
  { id: 2, label: 'Pagamento' },
  { id: 3, label: 'Quase lá!' },
]

const currentStep = ref(1)
const deliveryMethod = ref<'retirada' | 'entrega'>('retirada')
const address = ref('')
const paymentMethod = ref<'dinheiro' | 'cartao_credito' | 'cartao_debito' | 'pix'>('pix')
const needsChange = ref<boolean | null>(null)
const changeFor = ref('')

const total = computed(() => Math.max(props.subtotal - props.discount, 0) + (deliveryMethod.value === 'entrega' ? deliveryFee : 0))

const paymentLabel = computed(() => {
  if (paymentMethod.value === 'dinheiro') return 'Dinheiro'
  if (paymentMethod.value === 'cartao_credito') return 'Cartão de crédito'
  if (paymentMethod.value === 'cartao_debito') return 'Cartão de débito'
  return 'Pix'
})

const canAdvance = computed(() => {
  if (currentStep.value === 1) {
    return deliveryMethod.value === 'retirada' || address.value.trim().length > 0
  }
  if (currentStep.value === 2) {
    if (paymentMethod.value !== 'dinheiro') return true
    if (needsChange.value === null) return false
    return needsChange.value === false || changeFor.value.trim().length > 0
  }
  return true
})

watch(
  () => props.open,
  (isOpen) => {
    if (isOpen) {
      currentStep.value = 1
      deliveryMethod.value = 'retirada'
      address.value = ''
      paymentMethod.value = 'pix'
      needsChange.value = null
      changeFor.value = ''
    }
  },
)

function currency(value: number) {
  return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value)
}

function handleClose() {
  emit('close')
}

function confirmOrder() {
  emit('confirm', {
    items: props.items,
    deliveryMethod: deliveryMethod.value,
    address: address.value,
    deliveryFee: deliveryMethod.value === 'entrega' ? deliveryFee : 0,
    paymentMethod: paymentMethod.value,
    needsChange: needsChange.value ?? false,
    changeFor: changeFor.value,
    subtotal: props.subtotal,
    discount: props.discount,
    total: total.value,
    estimatedTime,
  })
}
</script>

<style scoped>
.checkout-overlay {
  position: fixed;
  inset: 0;
  z-index: 50;
  display: grid;
  place-items: center;
  background: rgba(24, 22, 22, 0.55);
  padding: 20px;
}

.checkout-modal {
  width: 100%;
  max-width: 480px;
  max-height: 88vh;
  display: flex;
  flex-direction: column;
  background: #fff;
  border: 1px solid var(--line);
}

.checkout-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 18px 20px;
  border-bottom: 1px solid var(--line);
}

.steps {
  display: flex;
  gap: 18px;
}

.step {
  display: flex;
  align-items: center;
  gap: 7px;
  color: #b3aead;
}

.step-index {
  width: 22px;
  height: 22px;
  display: grid;
  place-items: center;
  border-radius: 50%;
  border: 1px solid var(--line);
  font: 10px 'DM Mono', monospace;
}

.step-label {
  font-size: 11px;
  font-weight: 700;
}

.step.active {
  color: var(--ink);
}

.step.active .step-index {
  background: var(--red);
  border-color: var(--red);
  color: #fff;
}

.step.done {
  color: var(--ink);
}

.step.done .step-index {
  background: var(--ink);
  border-color: var(--ink);
  color: #fff;
}

.close-button {
  border: 0;
  background: none;
  font-size: 16px;
  color: var(--muted);
}

.checkout-body {
  padding: 22px 20px;
  overflow-y: auto;
}

.checkout-step h3 {
  margin-bottom: 4px;
  font-size: 16px;
  letter-spacing: -0.3px;
}

.summary-subtitle {
  margin-bottom: 16px;
  color: var(--muted);
  font-size: 12px;
}

.option-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin: 14px 0;
}

.option-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 13px 14px;
  border: 1px solid var(--line);
  cursor: pointer;
}

.option-card.selected {
  border-color: var(--red);
  background: #fdf3f2;
}

.option-card input {
  accent-color: var(--red);
}

.option-card div {
  flex: 1;
}

.option-card strong {
  display: block;
  font-size: 13px;
}

.option-card p {
  margin: 2px 0 0;
  color: var(--muted);
  font-size: 11px;
}

.option-price {
  font: 11px 'DM Mono', monospace;
  color: var(--muted);
}

.field {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: 14px;
  font-size: 12px;
  font-weight: 700;
}

.field input[type='text'] {
  padding: 11px 12px;
  border: 1px solid var(--line);
  outline: 0;
  font-size: 12px;
  font-weight: 400;
}

.field input[type='text']:focus {
  border-color: var(--red);
}

.toggle-row {
  display: flex;
  gap: 8px;
}

.toggle-row button {
  flex: 1;
  padding: 9px;
  border: 1px solid var(--line);
  background: #fff;
  font-size: 12px;
  font-weight: 700;
}

.toggle-row button.selected {
  border-color: var(--red);
  background: var(--red);
  color: #fff;
}

.summary-items {
  list-style: none;
  margin: 0 0 14px;
  padding: 0;
  border-bottom: 1px dashed var(--line);
}

.summary-items li {
  display: flex;
  justify-content: space-between;
  padding: 7px 0;
  font-size: 12px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  padding: 6px 0;
  color: var(--muted);
  font-size: 12px;
}

.summary-row strong {
  color: var(--ink);
}

.summary-row.total {
  margin-top: 4px;
  padding-top: 10px;
  border-top: 1px solid var(--line);
  font-weight: 800;
  font-size: 13px;
}

.checkout-footer {
  display: flex;
  gap: 10px;
  padding: 16px 20px;
  border-top: 1px solid var(--line);
}

.button {
  flex: 1;
  min-height: 44px;
  border: 0;
  font-size: 12px;
  font-weight: 800;
}

.button.primary {
  background: var(--red);
  color: #fff;
}

.button.primary:disabled {
  cursor: not-allowed;
  background: #c9c4c1;
}

.button.secondary {
  background: #fff;
  border: 1px solid var(--line);
  color: var(--ink);
}
</style>
