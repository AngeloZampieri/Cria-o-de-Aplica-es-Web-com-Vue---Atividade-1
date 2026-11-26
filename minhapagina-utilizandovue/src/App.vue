<script setup>
import { computed, reactive, ref, watch } from 'vue';

// Dados pessoais carregados do arquivo JSON.
const me = ref({});

// Estado do bate-papo e da notificação de nova mensagem.
const messages = reactive([]);
const newMessage = ref('');
const hasNewMessage = ref(false);

// Guia atualmente selecionada no menu.
const actualMenu = ref(0);
const menu = [
  'Sobre mim',
  'Formação acadêmica',
  'Experiência profissional',
  'Contato',
];

// Título que acompanha a guia selecionada.
const titlePage = computed(() => menu[actualMenu.value]);

// Seleciona uma guia do menu.
const changeMenu = (menuIndex) => {
  actualMenu.value = menuIndex;
};

// Adiciona a mensagem ao bate-papo e limpa o campo de digitação.
const sendMsg = () => {
  const text = newMessage.value.trim();

  if (!text) return;

  const date = new Date().toLocaleString('en-GB');
  messages.push({ text, date });
  newMessage.value = '';
};

// Fecha o aviso de nova mensagem.
const closeAlert = () => {
  hasNewMessage.value = false;
};

// Carrega os dados do perfil e das seções.
fetch('src/assets/aboutme.json')
  .then((response) => response.json())
  .then((data) => {
    me.value = data;
  });

// Exibe o aviso quando uma nova mensagem é adicionada.
watch(
  () => messages.length,
  (newLength, oldLength) => {
    if (newLength > oldLength) hasNewMessage.value = true;
  },
);
</script>

<template>
  <div class="layout">
    <!-- Menu fixo e título da guia atual. -->
    <header>
      <nav class="menu" aria-label="Navegação principal">
        <a
          v-for="(item, index) in menu"
          :key="`menuitem-${index}`"
          href="#"
          @click.prevent="changeMenu(index)"
        >
          {{ item }}
        </a>
      </nav>

      <h2 class="page-title">{{ titlePage }}</h2>
    </header>

    <!-- Aviso flutuante exibido após o envio de uma mensagem. -->
    <div v-if="hasNewMessage" class="alert" role="status">
      Nova mensagem disponível!
      <button type="button" aria-label="Fechar notificação" @click="closeAlert">
        X
      </button>
    </div>

    <!-- Conteúdo da guia selecionada. -->
    <main>
      <section v-show="actualMenu === 0" class="page-section">
        <img :src="me.photo" alt="Foto de Angelo" class="foto-perfil" />
        <h3>{{ me.name }}</h3>
        <p>{{ me.description }}</p>
      </section>

      <section v-show="actualMenu === 1" class="page-section">
        <article v-for="(item, index) in me.academic" :key="index" class="info-card">
          <h3>{{ item.course }}</h3>
          <p>{{ item.institution }}</p>
          <p>{{ item.year }}</p>
        </article>
      </section>

      <section v-show="actualMenu === 2" class="page-section">
        <article v-for="(item, index) in me.experience" :key="index" class="info-card">
          <h3>{{ item.position }}</h3>
          <p>{{ item.company }} — {{ item.year }}</p>
          <p>{{ item.description }}</p>
        </article>
      </section>

      <section v-show="actualMenu === 3" class="page-section">
        <p
          v-for="(item, index) in me.contact"
          :key="index"
          class="contact-item"
        >
          <span class="contact-icon" aria-hidden="true">
            {{ item.type === 'Email' ? '✉' : 'in' }}
          </span>

          <a
            v-if="item.type === 'LinkedIn'"
            :href="item.value"
            target="_blank"
            rel="noopener noreferrer"
          >
            {{ item.value }}
          </a>
          <a v-else-if="item.type === 'Email'" :href="`mailto:${item.value}`">
            {{ item.value }}
          </a>
          <span v-else>{{ item.value }}</span>
        </p>
      </section>
    </main>

    <!-- Bate-papo fixo; somente a lista de mensagens tem rolagem. -->
    <footer>
      <h2>Bate-papo</h2>

      <div class="chat-messages" aria-live="polite">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          class="msg__chat"
        >
          <p>{{ msg.text }}</p>
          <span>{{ msg.date }}</span>
        </div>
      </div>

      <form class="form__chat" @submit.prevent="sendMsg">
        <textarea
          v-model="newMessage"
          placeholder="Digite sua mensagem aqui"
          aria-label="Digite sua mensagem aqui"
        ></textarea>
        <button type="submit">Enviar</button>
      </form>
    </footer>
  </div>
</template>
