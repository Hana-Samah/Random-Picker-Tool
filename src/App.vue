<template>
  <div class="app">
    <div class="lang-switch">
      <button @click="toggleLang">{{ lang === 'en' ? 'عربي' : 'English' }}</button>
    </div>

    <h1>{{ t('title') }}</h1>

    <div v-if="!itemsEntered" class="input-section">
      <h2>{{ t('enterItems') }}</h2>
      <div class="input-group">
        <input
          v-model="currentItem"
          @keyup.enter="addItem"
          :placeholder="t('placeholder')"
          ref="itemInput"
        />
        <button @click="addItem">{{ t('addItem') }}</button>
      </div>

      <p v-if="error" class="error">{{ error }}</p>

      <div class="items-preview">
        <div v-for="(item, index) in items" :key="index" class="item">
          {{ item }}
          <button @click="removeItem(index)" class="remove-btn">×</button>
        </div>
      </div>

      <button
        @click="startRandomPicker"
        class="start-btn"
        :disabled="items.length < 2"
      >
        {{ t('start') }}
      </button>
    </div>

    <div v-else class="random-picker">
      <div class="items-list">
        <div
          v-for="(item, index) in items"
          :key="index"
          class="item"
          :class="{ selected: selectedIndex === index }"
        >
          {{ item }}
        </div>
      </div>

      <div v-if="selectedItem" class="selected-item">
        <h2>{{ t('selected') }}</h2>
        <p>{{ selectedItem }}</p>
      </div>

      <div class="controls">
        <button @click="pickRandomItem" class="pick-btn">
          {{ isPicking ? t('picking') : t('pick') }}
        </button>
        <button @click="resetAll" class="reset-btn">{{ t('restart') }}</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      lang: 'en',
      currentItem: '',
      items: [],
      itemsEntered: false,
      selectedItem: null,
      selectedIndex: null,
      isPicking: false,
      error: '',
      translations: {
        en: {
          title: 'Random Picker Tool',
          enterItems: 'Enter your items',
          placeholder: 'Type an item then click Add',
          addItem: 'Add',
          start: 'Start Random Selection',
          pick: 'Pick a Random Item',
          picking: 'Picking...',
          restart: 'Start Over',
          selected: 'Selected Item:',
          duplicate: 'This item already exists.'
        },
        ar: {
          title: 'أداة الاختيار العشوائي',
          enterItems: 'أدخل العناصر الخاصة بك',
          placeholder: 'اكتب عنصرًا ثم اضغط إضافة',
          addItem: 'إضافة',
          start: 'بدء الاختيار العشوائي',
          pick: 'اختر عنصرًا عشوائيًا',
          picking: 'جاري الاختيار...',
          restart: 'بدء من جديد',
          selected: 'العنصر المختار:',
          duplicate: 'هذا العنصر موجود بالفعل.'
        }
      }
    };
  },
  mounted() {
    const saved = localStorage.getItem('items');
    if (saved) this.items = JSON.parse(saved);
    this.$refs.itemInput?.focus();
  },
  methods: {
    t(key) {
      return this.translations[this.lang][key];
    },
    toggleLang() {
      this.lang = this.lang === 'en' ? 'ar' : 'en';
    },
    addItem() {
      this.error = '';
      const trimmed = this.currentItem.trim();
      if (!trimmed) return;

      if (this.items.includes(trimmed)) {
        this.error = this.t('duplicate');
        return;
      }

      this.items.push(trimmed);
      localStorage.setItem('items', JSON.stringify(this.items));
      this.currentItem = '';
      this.$refs.itemInput?.focus();
    },
    removeItem(index) {
      this.items.splice(index, 1);
      localStorage.setItem('items', JSON.stringify(this.items));
    },
    startRandomPicker() {
      if (this.items.length >= 2) {
        this.itemsEntered = true;
      }
    },
    pickRandomItem() {
      if (this.isPicking || this.items.length < 2) return;
      this.isPicking = true;
      this.selectedItem = null;
      let counter = 0;
      const total = 15;
      const interval = setInterval(() => {
        this.selectedIndex = Math.floor(Math.random() * this.items.length);
        counter++;
        if (counter >= total) {
          clearInterval(interval);
          this.selectedItem = this.items[this.selectedIndex];
          this.isPicking = false;
        }
      }, 100);
    },
    resetAll() {
      this.items = [];
      this.currentItem = '';
      this.itemsEntered = false;
      this.selectedItem = null;
      this.selectedIndex = null;
      localStorage.removeItem('items');
      this.$nextTick(() => {
        this.$refs.itemInput?.focus();
      });
    }
  }
};
</script>

<style scoped>
.app {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  text-align: center;
  font-family: 'Tahoma', sans-serif;
  background-color: #f9f9f9;
}

.lang-switch {
  display: flex;
  justify-content: flex-end;
}

.lang-switch button {
  background: #f8c8c8;
  border: 1px solid #f1a7a7;
  padding: 5px 10px;
  margin-bottom: 10px;
  cursor: pointer;
  color: #e74c3c;
  border-radius: 5px;
}

h1 {
  color: #f3a7c4;
  margin-bottom: 20px;
}

h2 {
  color: #e74c3c;
  margin-bottom: 15px;
}

.input-section {
  background: #f7e1e6;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.input-group {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

input {
  flex: 1;
  padding: 10px;
  border: 2px solid #f8c8c8;
  border-radius: 6px;
  font-size: 16px;
  background-color: #fff8f9;
}

button {
  padding: 10px 15px;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  cursor: pointer;
  background-color: #f3a7c4;
  color: white;
}

button:hover {
  background-color: #e74c3c;
}

.error {
  color: #e74c3c;
  margin: 5px 0;
}

.items-preview {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  margin: 10px 0 20px;
}

.item {
  background-color: #f2e6f2;
  padding: 8px 14px;
  border-radius: 15px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.remove-btn {
  background-color: transparent;
  color: #e74c3c;
  font-size: 18px;
  border: none;
  cursor: pointer;
}

.start-btn {
  background-color: #f3a7c4;
  color: white;
  padding: 12px 20px;
}

.start-btn:disabled {
  background-color: #e6b3b6;
  cursor: not-allowed;
}

.random-picker {
  margin-top: 20px;
}

.items-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.item.selected {
  background-color: #f8c8c8;
  transform: scale(1.05);
}

.selected-item {
  margin-top: 20px;
  padding: 20px;
  background-color: #fef4f6;
  border-radius: 8px;
  color: #e74c3c;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 30px;
}

.reset-btn {
  background-color: #e74c3c;
  color: white;
  padding: 12px 20px;
}

.reset-btn:hover {
  background-color: #e63946;
}
</style>
