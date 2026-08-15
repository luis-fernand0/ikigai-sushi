<template>
  <header class="site-header">
    <a class="brand" href="#inicio" aria-label="IKIGAI início">
      <img src="/logo-ikigai.jpg" alt="Logo IKIGAI">
      <strong>IKIGAI</strong>
    </a>

    <label class="search">
      <span>⌕</span>
      <input type="search" placeholder="Buscar no cardápio" />
    </label>

    <nav aria-label="Navegação principal">
      <a class="active" href="#inicio">
        ⌂ <span>Início</span>
      </a>
      <a href="#cardapio">
        ▤ <span>Cardápio</span>
      </a>
      <a href="#pedidos">
        ▣ <span>Pedidos</span>
      </a>
      <a href="#perfil">
        ♙ <span>Perfil</span>
      </a>
    </nav>
  </header>

  <main id="inicio">
    <section class="hero">
      <img
        src="/capa-ikigai.jpg"
        alt="Seleção de sushi fresco em uma mesa" />

      <div class="hero-shade"></div>
      
      <div class="hero-copy">
        <p class="eyebrow">DELIVERY</p>

        <h1>O sabor que encontra<br /><em>seu propósito.</em></h1>

        <p>Uma experiência japonesa criada para marcar o seu momento.</p>

        <a class="button primary" href="#cardapio">
          Ver cardápio <span>→</span>
        </a>
      </div>

      <p class="vertical-word">IKIGAI / DOURADOS</p>
    </section>

    <div class="layout-shell">
      <div class="content-column">
        <section class="restaurant-info" aria-label="Informações do restaurante">
          <div class="round-logo">
            <img src="/logo-ikigai.jpg" alt="Logo IKIGAI">
          </div>

          <div class="restaurant-copy">
            <div class="title-line">
              <h2>Ikigai Sushi</h2>
              <span class="open-status">Aberto agora</span>
            </div>

            <p class="rating">
              ★ 5.0 <span>(1.240 avaliações)</span>
            </p>

            <p class="details">
              Rua Floriano Peixoto, 574 · Centro · <strong>35-45 min</strong>
            </p>

          </div>

          <button class="share" aria-label="Compartilhar restaurante">
            ↗
          </button>
        </section>

        <section class="promotions" aria-label="Promoções em destaque">
          <article class="promo-card dark-promo">
            <img
              src="https://images.unsplash.com/photo-1617196034183-421b4040f6c0?auto=format&fit=crop&w=900&q=85"
              alt="Combinado de sushi" />

            <div>
              <p>PARA COMPARTILHAR</p>

              <h3>Combinados<br />especiais</h3>

              <a href="#cardapio">Explorar 
                <span>→</span>
              </a>
            </div>
          </article>

          <article class="promo-card red-promo"><img
              src="https://images.unsplash.com/photo-1580822184713-fc5400e7fe10?auto=format&fit=crop&w=900&q=85"
              alt="Prato japonês especial" />
            <div>
              <p>SELEÇÃO DA CASA</p>

              <h3>Experiência<br />Ikigai</h3>
              
              <a href="#cardapio">
                Peça agora <span>→</span>
              </a>
            </div>
          </article>
        </section>

        <section id="cardapio" class="menu-section">
          <div class="section-heading">
            <div>
              <p class="eyebrow">ESCOLHA SEU FAVORITO</p>
              <h2>Cardápio</h2>
            </div>
            
            <span>{{ totalItems }} {{ totalItems === 1 ? 'item' : 'itens' }} na sacola</span>
          </div>
          <div class="categories" role="tablist" aria-label="Categorias do cardápio">
            <button v-for="category in categories" :key="category" :class="{ selected: activeCategory === category }"
              @click="activeCategory = category"
              >
              {{ category }}
            </button>
          </div>

          <div class="product-grid">
            <article v-for="product in displayedProducts" :key="product.id" class="product-card">
              <img :src="product.image" :alt="product.name" />

              <div class="product-details">
                <h3>{{ product.name }}</h3>

                <p>{{ product.description }}</p>

                <div>
                  <strong>{{ currency(product.price) }}</strong>

                  <button :aria-label="`Adicionar ${product.name}`"
                    @click="addToCart(product)"
                    >
                    +
                  </button>
                </div>
              </div>
            </article>

            <p v-if="!displayedProducts.length" class="empty-category">Novas opções em breve nesta categoria.</p>
          </div>
        </section>
      </div>

      <aside class="cart" id="pedidos">
        <div class="cart-top">
          <div>
            <p class="eyebrow">SEU PEDIDO</p>

            <h2>Sua sacola</h2>
          </div>
          
          <span class="cart-count">{{ totalItems }}</span>
        </div>
        <div v-if="cart.length" class="cart-items">
          <article v-for="item in cart" :key="item.product.id" class="cart-item">
            <img :src="item.product.image" :alt="item.product.name" />

            <div>
              <h3>{{ item.product.name }}</h3>

              <p>{{ currency(item.product.price) }}</p>

              <div class="quantity">
                <button @click="adjustQuantity(item.product.id, -1)" >
                  −
                </button>

                <span>
                  {{ item.quantity }}
                </span>
                
                <button @click="adjustQuantity(item.product.id, 1)">
                  +
                </button>
              </div>
            </div>
          </article>
        </div>

        <div v-else class="empty-cart">
          <span>◌</span>

          <h3>Sua sacola está vazia</h3>

          <p>Escolha seus favoritos e eles aparecem aqui.</p>
        </div>

        <div class="cart-summary">
          <label class="coupon">
            ⌁ <input v-model="couponCode" placeholder="Tem um cupom?" @keyup.enter="applyCoupon" />

            <button @click="applyCoupon">{{ appliedCoupon ? 'Aplicado' : 'Aplicar' }}</button>
          </label>

          <p v-if="couponError" class="coupon-error">{{ couponError }}</p>

          <p><span>Subtotal</span><strong>{{ currency(subtotal) }}</strong></p>

          <p><span>Desconto</span><strong>{{ discount > 0 ? `- ${currency(discount)}` : currency(0) }}</strong></p>

          <p><span>Taxa de entrega</span><strong>Grátis</strong></p>

          <p class="cart-total"><span>Total</span><strong>{{ currency(total) }}</strong></p>
          
          <button class="button checkout" :disabled="!cart.length" @click="isCheckoutOpen = true">
            {{ cart.length ? 'Finalizar pedido' : 'Adicione itens para pedir' }} <span>→</span>
          </button>
        </div>
      </aside>
    </div>
  </main>

  <CheckoutModal
    :open="isCheckoutOpen"
    :items="cart"
    :subtotal="subtotal"
    :discount="discount"
    @close="isCheckoutOpen = false"
    @confirm="handleOrderConfirmed" />
</template>

<script setup lang="ts">
  import { computed, ref } from 'vue'
  import { produtos } from './utils/cardapio-fake.js'
  import { cupons } from './utils/cupons-fakes.js'
  import CheckoutModal from './components/CheckoutModal.vue'

  type Product = {
    id: number;
    name: string;
    description: string;
    price: number;
    image: string;
    category: string
  }

  const categories = ['Promoções', 'Combinados', 'Sushi', 'Sashimi', 'Temaki', 'Entradas', 'Pratos quentes', 'Bebidas'];

  const activeCategory = ref(categories[0]);

  const cart = ref<{ product: Product; quantity: number }[]>([]);

  const isCheckoutOpen = ref(false);

  const couponCode = ref('');

  const appliedCoupon = ref<string | null>(null);

  const couponError = ref('');

  const products: Product[] = produtos;

  const displayedProducts = computed(() => {
    if (activeCategory.value === categories[0]) {
      return products;
    } else {
      return products.filter((product) => product.category === activeCategory.value);
    }
  })

  const subtotal = computed(() => cart.value.reduce((sum, item) => sum + item.product.price * item.quantity, 0));

  const discount = computed(() => appliedCoupon.value ? 15 : 0);

  const total = computed(() => Math.max(subtotal.value - discount.value, 0));

  const totalItems = computed(() => cart.value.reduce((sum, item) => sum + item.quantity, 0));

  function addToCart(product: Product) {
    const existing = cart.value.find((item) => item.product.id === product.id);

    if (existing) existing.quantity += 1; else cart.value.push({ product, quantity: 1 })
  }

  function adjustQuantity(productId: number, amount: number) {
    const item = cart.value.find((cartItem) => cartItem.product.id === productId);

    if (!item) return;
    
    item.quantity += amount;
    
    if (item.quantity === 0) cart.value = cart.value.filter((cartItem) => cartItem.product.id !== productId);
  }

  function currency(value: number) {
    return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value)
  }

  function applyCoupon() {
    const code = couponCode.value.trim().toUpperCase();

    if (!code) return;

    if (cupons.includes(code)) {
      appliedCoupon.value = code;
      couponError.value = '';
    } else {
      appliedCoupon.value = null;
      couponError.value = 'Cupom inválido';
    }
  }

  function handleOrderConfirmed() {
    isCheckoutOpen.value = false;
    cart.value = [];
    couponCode.value = '';
    appliedCoupon.value = null;
    couponError.value = '';
  }
</script>
