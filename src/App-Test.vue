<!-- App.vue (Refactored) -->
<script setup>
import { ref, computed, onMounted, watch } from "vue";
import { Icon } from "@iconify/vue";

// ============================================
// REACTIVE DATA DECLARATIONS
// ============================================

// Page & Authentication State (~ Tiean ~)
const currentPage = ref("login");
const nickname = ref("");

const form = ref({
  title: "",
  date: new Date().toLocaleDateString("en-CA"),
  start: "09:00",
  end: "10:00",
});

// Events State
const events = ref(JSON.parse(localStorage.getItem("events") || "[]"));

// Cart State
const cart = ref(JSON.parse(localStorage.getItem("cart") || "[]"));

// FilterType & Search State  (~ Tiean ~)
const selectedType = ref(null);
const searchQuery = ref("");

// Filter price
const sortMode = ref("");
const showDropdown = ref(false);
const priceRange = ref({ min: 0, max: 100 });

// Items Data with Favorites
const savedFavorites = JSON.parse(localStorage.getItem("favorites") || "[]");
const items = ref(
  [
    {
      foodType: "Dessert",
      name: "Blueberry Cheesecake",
      description:
        "ชีสเค้กเนื้อเนียนนุ่ม โรยหน้าด้วยบลูเบอร์รี่ซอสเปรี้ยวหวานตัดกันอย่างลงตัว",
      img: "/src/assets/image/Blueberry-Cheesecake.jpg",
      price: 12,
      stock: 10,
    },
    {
      foodType: "Food",
      name: "Spaghetti Carbonara",
      description:
        "สปาเกตตีเส้นเหนียวนุ่ม คลุกเคล้าซอสครีมชีสและไข่แดง โรยด้วยเบคอนกรอบและพาเมซานชีส",
      img: "/src/assets/image/Spaghetti-Carbonara.jpg",
      price: 18,
      stock: 10,
    },
    {
      foodType: "Drink",
      name: "Iced Caramel Macchiato",
      description:
        "กาแฟเอสเปรสโซ่เข้มข้น ราดด้วยนมสดและคาราเมลซอสหอมหวาน ดื่มแล้วสดชื่น มีความกลมกล่อมลงตัว",
      img: "/src/assets/image/Iced-Caramel-Macchiato.jpg",
      price: 7,
      stock: 10,
    },
    {
      foodType: "Dessert",
      name: "Strawberry Shortcake",
      description:
        "เค้กนุ่มเบา สอดไส้ครีมสดและสตรอว์เบอร์รี่สด รสหวานอมเปรี้ยวลงตัว เหมาะกับทุกโอกาส",
      img: "/src/assets/image/Strawberry-Shortcake.jpg",
      price: 11,
      stock: 10,
    },
  ].map((item) => ({
    ...item,
    isFavourite: savedFavorites.includes(item.name),
  }))
);

// ============================================
// COMPUTED PROPERTIES
// ============================================

// Authentication & Validation
const isLoginValid = () => nickname.value.trim().length > 0; // (~ Tiean ~)

const isEventFormValid = computed(() => {
  return (
    form.value.title.trim() &&
    form.value.date &&
    form.value.start &&
    form.value.end &&
    form.value.end > form.value.start
  );
});

// Cart Computations
const cartCount = computed(() =>
  cart.value.reduce((sum, item) => sum + item.quantity, 0)
);
const cartTotal = computed(() =>
  cart.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
);

// Items & Filtering.  (~ Tiean ~)
const foodTypes = computed(() => [
  ...new Set(items.value.map((item) => item.foodType)),
]);
const favourites = computed(() =>
  items.value.filter((item) => item.isFavourite)
);
const favouriteCount = computed(() => favourites.value.length);

const filteredItems = computed(() => {
  const query = searchQuery.value.toLowerCase();

  let filtered = items.value
    // Filter by food type
    .filter(
      (item) => !selectedType.value || item.foodType === selectedType.value
    )
    // Search filter
    .filter(
      (item) =>
        item.name.toLowerCase().includes(query) ||
        item.description.toLowerCase().includes(query) ||
        item.foodType.toLowerCase().includes(query)
    )
    // Price range filter
    .filter(
      (item) =>
        item.price >= priceRange.value.min && item.price <= priceRange.value.max
    );

  // Sort favorites first
  filtered.sort((a, b) => Number(b.isFavourite) - Number(a.isFavourite));

  // Sort by name if specified
  if (sortMode.value === "asc") {
    filtered.sort((a, b) => a.name.localeCompare(b.name));
  } else if (sortMode.value === "desc") {
    filtered.sort((a, b) => b.name.localeCompare(a.name));
  }

  return filtered;
});

// Events Grouping
const eventsByDate = computed(() => {
  const grouped = {};
  events.value.forEach((event) => {
    if (!grouped[event.date]) {
      grouped[event.date] = [event];
    } else {
      grouped[event.date].push(event);
    }
  });
  return Object.entries(grouped).sort(([dateA], [dateB]) =>
    dateA.localeCompare(dateB)
  );
});

// ============================================
// WATCHERS
// ============================================

// Persist Events to LocalStorage
watch(
  () => JSON.stringify(events.value),
  () => localStorage.setItem("events", JSON.stringify(events.value)),
  { immediate: true }
);

// Persist Cart to LocalStorage
watch(
  () => JSON.stringify(cart.value),
  () => localStorage.setItem("cart", JSON.stringify(cart.value)),
  { immediate: true }
);

// ============================================
// UTILITY FUNCTIONS
// ============================================

function persistFavorites() {
  const favoriteNames = items.value
    .filter((item) => item.isFavourite)
    .map((item) => item.name);
  localStorage.setItem("favorites", JSON.stringify(favoriteNames));
}

// ============================================
// AUTHENTICATION FUNCTIONS. (~ Tiean ~)
// ============================================

function login() {
  if (nickname.value.trim() !== "") {
    localStorage.setItem("nickname", nickname.value.trim());
    currentPage.value = "home";
  }
}

function logout() {
  localStorage.removeItem("nickname");
  nickname.value = "";
  currentPage.value = "login";
}

// ============================================
// EVENT MANAGEMENT FUNCTIONS
// ============================================

function addEvent() {
  if (!isEventFormValid.value) return null;

  const newEvent = {
    title: form.value.title.trim(),
    date: form.value.date,
    start: form.value.start,
    end: form.value.end,
  };

  events.value.push(newEvent);
  events.value.sort((a, b) =>
    (a.date + a.start).localeCompare(b.date + b.start)
  );

  form.value.title = "";
  return newEvent;
}

function removeEvent(index) {
  events.value.splice(index, 1);
}

// ============================================
// CART MANAGEMENT FUNCTIONS
// ============================================

function addToCart(item, quantity = 1) {
  const existingItem = cart.value.find(
    (cartItem) => cartItem.name === item.name
  );
  if (existingItem) {
    existingItem.quantity += quantity;
  } else {
    cart.value.push({
      name: item.name,
      price: item.price,
      img: item.img,
      foodType: item.foodType,
      quantity: quantity,
    });
  }
}

function increaseCartItem(index) {
  cart.value[index].quantity += 1;
}
function decreaseCartItem(index) {
  if (cart.value[index].quantity > 1) {
    cart.value[index].quantity -= 1;
  }
}
function removeFromCart(index) {
  cart.value.splice(index, 1);
}
function clearAllCart() {
  cart.value = [];
}

// ============================================
// FILTER & SEARCH FUNCTIONS
// ============================================
//(~ Tiean ~)
function selectType(type) {
  selectedType.value = type;
}

function toggleDropdown() {
  showDropdown.value = !showDropdown.value;
}

function applyPriceRange() {
  showDropdown.value = false;
}

function sortAZ() {
  sortMode.value = "asc";
  showDropdown.value = false;
}

function sortZA() {
  sortMode.value = "desc";
  showDropdown.value = false;
}

const alreadySuggested = ref([]);

function suggestRandomItem() {
  const pool = filteredItems.value.filter(
    (item) => !alreadySuggested.value.includes(item.name)
  );

  if (pool.length === 0) {
    alert("สุ่มครบทุกเมนูแล้ว! รีเฟรชหรือกดรีเซ็ตเพื่อเริ่มใหม่");
    alreadySuggested.value = [];
    return;
  }

  const randomIndex = Math.floor(Math.random() * pool.length);
  const item = pool[randomIndex];

  alreadySuggested.value.push(item.name);
  alert(`🎉 เมนูที่แนะนำ: ${item.name}\nราคา: ฿${item.price}`);
}

function toggleFavourite(item) {
  item.isFavourite = !item.isFavourite;
  persistFavorites();
}

//recentlyViewed
const recentlyViewed = ref([]);

onMounted(() => {
  const recent = localStorage.getItem("recentlyViewed"); 
  if (recent) {
    recentlyViewed.value = JSON.parse(recent); 
  } else {
    recentlyViewed.value = []; 
  }
});

function viewRecentItem(item) {
  let newList = [];
  newList.push(item); 

  for (let i = 0; i < recentlyViewed.value.length; i++) { 
    let oldItem = recentlyViewed.value[i];

    if (oldItem.name != item.name) {
      newList.push(oldItem);
    }
  }

  newList = newList.slice(0, 3); 
  recentlyViewed.value = newList;
  localStorage.setItem("recentlyViewed", JSON.stringify(recentlyViewed.value)); 
}

// ============================================
// LIFECYCLE HOOKS
// ============================================

onMounted(() => {
  // Close dropdown when clicking outside
  document.addEventListener("click", (event) => {
    const dropdown = document.getElementById("dropdown-toggle");
    if (dropdown && !dropdown.contains(event.target)) {
      showDropdown.value = false;
    }
  });
});
</script>

<template>
  <div class="flex h-screen w-screen overflow-hidden">
    <!-- ============================================ -->
    <!-- LOGIN PAGE -->
    <!-- ============================================ -->
    <div
      v-if="currentPage === 'login'"
      class="min-h-screen w-full flex items-center justify-center bg-gradient-to-br from-orange-50 to-white p-4"
    >
      <div class="w-full max-w-sm">
        <div
          class="bg-white rounded-2xl shadow-lg border border-orange-200 p-6 sm:p-8"
        >
          <!-- Header -->
          <h1 class="text-2xl font-bold text-orange-600 text-center mb-6">
            Log in
          </h1>

          <!-- Nickname -->
          <input
            v-model="nickname"
            type="text"
            placeholder="Enter your nickname"
            class="text-black w-full border border-orange-300 p-3 rounded-xl outline-none focus:border-orange-500 focus:ring-4 focus:ring-orange-100 transition mb-4"
          />

          <!-- ปุ่มเข้าระบบ -->
          <button
            @click="login"
            :disabled="!isLoginValid()"
            class="w-full bg-orange-500 hover:bg-orange-600 active:scale-[.99] text-white px-4 py-3 rounded-xl font-medium shadow-md shadow-orange-200 disabled:opacity-50 disabled:cursor-not-allowed transition"
          >
            Enter
          </button>
        </div>
      </div>
    </div>

    <!-- ============================================ -->
    <!-- HOME PAGE -->
    <!-- ============================================ -->
    <div
      v-else-if="currentPage === 'home'"
      class="flex flex-col flex-1 overflow-hidden"
    >
      <!-- Header -->
      <header
        class="h-20 bg-white shadow-md flex items-center justify-between px-8 flex-shrink-0"
      >
        <div
          class="flex justify-center h-20 w-30 cursor-pointer"
          @click="currentPage = 'home'"
        >
          <img src="./assets/MarketLinkLogo.png" alt="Logo" />
        </div>

        <!-- Search + Filter -->
        <div class="flex-1 max-w-4xl mx-10 flex items-center gap-2 relative">
          <!-- Search Box -->
          <input
            type="text"
            placeholder="Search"
            v-model="searchQuery"
            class="w-full px-3 py-2 rounded-full border border-gray-400 text-black focus:outline-none focus:ring-2 focus:ring-orange-400"
          />

          <!-- ปุ่ม ... (Dropdown) -->
          <div class="relative" id="dropdown-toggle">
            <button
              @click="toggleDropdown"
              class="px-3 py-2 bg-white border rounded-full border-gray-300 hover:bg-gray-100 text-sm text-black"
            >
              ⋮
            </button>

            <div
              v-if="showDropdown"
              class="absolute right-0 mt-2 w-60 bg-white border border-gray-300 rounded-lg shadow-lg z-10 p-3 space-y-2"
            >
              <div class="px-4 py-2 text-sm">
                <div class="flex space-x-2 items-center mb-2">
                  <label class="text-xs text-black">Min:</label>
                  <input
                    v-model.number="priceRange.min"
                    type="number"
                    class="w-14 px-1 py-0.5 text-black rounded"
                  />
                  <label class="text-xs text-black">Max:</label>
                  <input
                    v-model.number="priceRange.max"
                    type="number"
                    class="w-14 px-1 py-0.5 text-black rounded"
                  />
                </div>
                <button
                  @click="applyPriceRange"
                  class="block w-full text-center mt-1 px-3 py-1 bg-orange-500 rounded text-black hover:bg-orange-600 text-xs"
                >
                  Apply Range
                </button>
              </div>
              <button
                @click="sortAZ"
                class="block w-full text-left px-4 py-2 text-sm hover:bg-gray-100 text-black"
              >
                A-Z
              </button>
              <button
                @click="sortZA"
                class="block w-full text-left px-4 py-2 text-sm hover:bg-gray-100 text-black"
              >
                Z-A
              </button>
            </div>
          </div>

          <!-- ปุ่มสุ่มเมนู -->
          <button
            @click="suggestRandomItem"
            class="ml-2 px-2 py-1 rounded-full border border-gray-300 hover:bg-gray-100 text-xs text-black"
          >
            Random Menu!
          </button>
        </div>

        <!-- Navigation Icons + User Info -->
        <div class="flex items-center space-x-4">
          <!-- Calendar Icon -->
          <Icon
            icon="mdi:calendar-outline"
            width="24"
            height="24"
            color="black"
            class="cursor-pointer"
            @click="currentPage = 'calendar'"
          />

          <!-- Cart Icon with Badge -->
          <button class="relative" @click="currentPage = 'cart'" title="Cart">
            <Icon
              icon="mdi:cart-outline"
              width="28"
              height="28"
              color="black"
            />
            <span
              v-if="cartCount > 0"
              class="absolute -top-1 -right-2 bg-orange-500 text-white text-[10px] px-1.5 py-0.5 rounded-full"
            >
              {{ cartCount }}
            </span>
          </button>

          <!-- Favourite Icon with Badge -->
          <button
            class="relative"
            @click="currentPage = 'favourites'"
            title="Favourites"
          >
            <Icon
              icon="fluent-emoji-high-contrast:star"
              width="26"
              height="26"
              color="black"
            />
            <span
              v-if="favouriteCount > 0"
              class="absolute -top-1 -right-2 bg-orange-500 text-white text-[10px] px-1.5 py-0.5 rounded-full"
            >
              {{ favouriteCount }}
            </span>
          </button>

          <!-- User Avatar & Info -->
          <div
            class="w-10 h-10 rounded-full bg-blue-500 flex items-center justify-center text-white font-bold"
          >
            {{ nickname.charAt(0).toUpperCase() }}
          </div>
          <span class="font-semibold text-black">{{ nickname }}</span>

          <!-- Logout Button -->
          <button
            class="ml-2 px-3 py-1 rounded-full border border-gray-300 hover:bg-gray-100 text-sm text-black"
            @click="logout"
            title="Logout"
          >
            Logout
          </button>
        </div>
      </header>

      <!-- Main Content -->
      <main class="flex-1 overflow-y-auto px-8 py-5 bg-gray-50">
        <!-- Category Filter Buttons -->
        <div class="flex py-3 space-x-2 flex-wrap text-black items-center">
          <p class="text-black text-lg font-semibold pr-5">Category</p>

          <button
            @click="selectType(null)"
            class="rounded-2xl px-3 py-1 shadow-md transition-colors duration-300 text-xs"
            :class="
              selectedType === null
                ? 'bg-orange-500 text-white hover:bg-orange-600'
                : 'bg-white text-black hover:bg-orange-200'
            "
          >
            All
          </button>

          <button
            v-for="type in foodTypes"
            :key="type"
            @click="selectType(type)"
            class="rounded-2xl px-3 py-1 shadow-md transition-colors duration-300 text-xs"
            :class="
              selectedType === type
                ? 'bg-orange-500 text-white hover:bg-orange-600'
                : 'bg-white text-black hover:bg-orange-200'
            "
          >
            {{ type }}
          </button>
        </div>

        <!-- Product Cards Grid -->
        <div class="flex flex-wrap gap-4">
          <div
            v-for="item in filteredItems"
            :key="item.name"
            class="bg-[#f9f8f6] rounded-2xl p-3 shadow-md overflow-hidden hover:shadow-lg transition w-48 sm:w-56 md:w-64 lg:w-72 relative"
          >
            <!-- Favorite Button -->
            <button
              class="absolute top-4 right-4"
              @click="toggleFavourite(item)"
            >
              <Icon
                icon="fluent-emoji-high-contrast:star"
                width="30"
                height="30"
                :color="item.isFavourite ? 'orange' : 'white'"
              />
            </button>

            <!-- Product Image -->
            <div
              class="cursor-pointer w-full h-[220px]"
              @click="viewRecentItem(item)"
            >
              <img
                :src="item.img"
                alt=""
                class="w-full h-full object-cover rounded-xl"
              />
            </div>

            <!-- Product Info -->
            <div class="p-2">
              <h2 class="text-xs font-semibold text-black">{{ item.name }}</h2>
              <p class="text-gray-600 text-[10px] mt-1">
                {{ item.description }}
              </p>

              <!-- Price & Add to Cart Button -->
              <div class="mt-2 flex items-center justify-between gap-2">
                <span
                  class="text-green-600 font-semibold text-xs whitespace-nowrap"
                >
                  ฿{{ item.price }}
                </span>
                <button
                  class="px-3 py-1.5 rounded-full bg-orange-500 text-white font-medium hover:bg-orange-600 text-xs whitespace-nowrap shrink-0"
                  @click="addToCart(item, 1)"
                >
                  Add to Cart
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- Flash Sale Section -->
        <div
          class="mt-6 mb-4 flex items-center justify-between bg-[#d6833e] p-3 rounded-2xl shadow-lg"
        >
          <h1 class="text-white text-2xl font-extrabold">🔥 Flash Sale</h1>
          <span
            class="text-white text-sm font-medium bg-white/30 px-3 py-1 rounded-full"
          >
            วันนี้เท่านั้น!
          </span>
        </div>

        <!-- Promotion Section -->
        <div class="mt-8 p-3 bg-gray-100 rounded-2xl shadow-md">
          <h2 class="text-lg font-bold text-gray-800 mb-4">โปรโมชั่นล่าสุด</h2>
          <div class="flex gap-4 p-1">
            <div class="bg-white rounded-xl shadow p-4 w-48 text-center">
              <h2 class="font-bold text-lg text-black">Pizza with Peperoni</h2>
              <p class="text-black text-sm mt-1">14-20 minutes</p>
              <p class="text-green-500 font-semibold mt-1">$12</p>
            </div>
            <div class="bg-white rounded-xl shadow p-4 w-48 text-center">
              <h2 class="font-bold text-lg text-black">Pizza with Cheese</h2>
              <p class="text-black text-sm mt-1">16-25 minutes</p>
              <p class="text-green-500 font-semibold mt-1">$14</p>
            </div>
          </div>
        </div>

        <!-- Recently Viewed -->
        <div class="mt-8 p-3 bg-gray-100 rounded-2xl shadow-md">
          <h2 class="text-lg font-bold text-gray-800 mb-4">ดูล่าสุด</h2>
          <div
            v-if="recentlyViewed.length > 0"
            class="flex gap-4 overflow-x-auto p-1"
          >
            <div
              v-for="item in recentlyViewed"
              :key="item.name"
              class="bg-white rounded-xl shadow p-4 w-48 text-center flex-shrink-0"
            >
              <img
                :src="item.img"
                alt=""
                class="w-full h-28 object-cover rounded-md mb-2"
              />
              <h2 class="font-bold text-sm text-black truncate">
                {{ item.name }}
              </h2>
              <p class="text-green-500 font-semibold mt-1">฿{{ item.price }}</p>
            </div>
          </div>
          <p v-else class="text-gray-500 text-sm">ยังไม่มีสินค้าที่ดูล่าสุด</p>
        </div>
      </main>
    </div>

    <!-- ============================================ -->
    <!-- CALENDAR PAGE -->
    <!-- ============================================ -->
    <div
      v-else-if="currentPage === 'calendar'"
      class="p-6 bg-white rounded-xl shadow-md w-full text-black"
    >
      <div class="flex items-center justify-between mb-4">
        <h1 class="text-2xl font-bold">Market Days</h1>
        <button
          class="px-3 py-1.5 text-sm border rounded-full hover:bg-gray-100"
          @click="currentPage = 'home'"
        >
          Back to Home
        </button>
      </div>

      <!-- Event Form -->
      <div class="bg-gray-50 p-4 rounded-xl border mb-6">
        <p v-if="!isEventFormValid" class="text-xs text-red-600 mt-2">
          กรุณากรอกข้อมูลให้ครบ
        </p>

        <div class="grid gap-3 sm:grid-cols-5">
          <div class="sm:col-span-2">
            <label class="text-xs text-gray-600">Title</label>
            <input
              v-model="form.title"
              type="text"
              placeholder="เช่น โปรฯ บ่าย / ไลฟ์สด"
              class="w-full border rounded px-2 py-1"
            />
          </div>
          <div>
            <label class="text-xs text-gray-600">Date</label>
            <input
              v-model="form.date"
              type="date"
              class="w-full border rounded px-2 py-1"
            />
          </div>
          <div>
            <label class="text-xs text-gray-600">Start</label>
            <input
              v-model="form.start"
              type="time"
              class="w-full border rounded px-2 py-1"
            />
          </div>
          <div>
            <label class="text-xs text-gray-600">End</label>
            <input
              v-model="form.end"
              type="time"
              class="w-full border rounded px-2 py-1"
            />
          </div>
        </div>

        <div class="mt-3">
          <button
            @click="addEvent"
            :disabled="!isEventFormValid"
            class="px-4 py-2 rounded-full bg-black text-white hover:bg-gray-800 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            + Add
          </button>
        </div>
      </div>

      <!-- Events List -->
      <div v-if="eventsByDate.length === 0" class="text-gray-600">
        ยังไม่มีอีเวนต์ในตาราง
      </div>

      <div v-else class="space-y-5">
        <div
          v-for="[date, eventList] in eventsByDate"
          :key="date"
          class="border rounded-xl overflow-hidden"
        >
          <div class="px-4 py-2 bg-gray-100 font-semibold">
            {{
              new Date(date).toLocaleDateString(undefined, {
                weekday: "short",
                day: "2-digit",
                month: "short",
                year: "numeric",
              })
            }}
          </div>
          <div class="divide-y">
            <div
              v-for="event in eventList"
              :key="event.id"
              class="px-4 py-3 flex items-center justify-between"
            >
              <div>
                <div class="font-semibold">{{ event.title }}</div>
                <div class="text-sm text-gray-600">
                  {{ event.start }}–{{ event.end }}
                </div>
              </div>
              <button
                @click="removeEvent(index)"
                class="px-4 py-2 rounded-full bg-[#c52d2d] text-white hover:bg-[#931818]"
              >
                Remove
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ============================================ -->
    <!-- CART PAGE -->
    <!-- ============================================ -->
    <div
      v-else-if="currentPage === 'cart'"
      class="flex-1 overflow-y-auto px-6 py-6 bg-gray-50"
    >
      <div class="flex items-center justify-between mb-4">
        <h1 class="text-2xl font-bold text-black">My Cart ({{ cartCount }})</h1>
        <div class="flex gap-2">
          <button
            class="px-3 py-1.5 text-sm border rounded-full hover:bg-gray-100 text-black border-gray-300"
            @click="currentPage = 'home'"
          >
            Back to Home
          </button>
          <button
            v-if="cart.length"
            class="px-3 py-1.5 text-sm border border-red-300 text-red-600 rounded-full hover:bg-red-50"
            @click="clearAllCart()"
          >
            Clear Cart
          </button>
        </div>
      </div>

      <div v-if="cart.length === 0" class="text-black">
        ยังไม่มีสินค้าในตะกร้า
      </div>

      <div v-else class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <!-- Cart Items List -->
        <div class="lg:col-span-2 space-y-3">
          <div
            v-for="(cartItem, index) in cart"
            :key="cartItem.name"
            class="bg-white rounded-xl p-3 shadow flex items-center gap-3"
          >
            <img
              :src="cartItem.img"
              alt=""
              class="w-20 h-20 object-cover rounded-lg"
            />

            <div class="flex-1">
              <div class="font-semibold text-black">{{ cartItem.name }}</div>
              <div class="text-xs text-gray-500">{{ cartItem.foodType }}</div>
              <div class="text-black text-sm font-semibold mt-1">
                ฿{{ cartItem.price }}
              </div>
            </div>

            <!-- Quantity Controls -->
            <div class="flex items-center gap-2">
              <button
                class="w-7 h-7 border text-black rounded-full"
                @click="decreaseCartItem(index)"
              >
                −
              </button>
              <div class="w-8 text-center text-black">
                {{ cartItem.quantity }}
              </div>
              <button
                class="w-7 h-7 border text-black rounded-full"
                @click="increaseCartItem(index)"
              >
                +
              </button>
            </div>

            <!-- Subtotal -->
            <div class="w-20 text-right font-semibold text-black">
              ฿{{ (cartItem.price * cartItem.quantity).toFixed(2) }}
            </div>

            <!-- Remove Button -->
            <button
              class="ml-2 text-red-500 hover:text-red-600 text-sm underline"
              @click="removeFromCart(index)"
            >
              Remove
            </button>
          </div>
        </div>

        <!-- Order Summary -->
        <aside class="bg-white rounded-2xl p-4 shadow h-fit">
          <h2 class="text-lg font-bold text-black mb-2">Order Summary</h2>
          <div class="flex justify-between text-sm mb-1 text-black">
            <span>Items</span>
            <span>{{ cartCount }}</span>
          </div>
          <div class="flex justify-between text-sm mb-3 text-black">
            <span>Total</span>
            <span>฿{{ cartTotal.toFixed(2) }}</span>
          </div>
          <button
            :disabled="cart.length === 0"
            class="w-full px-4 py-2 rounded-full bg-black text-white font-semibold disabled:opacity-50 disabled:cursor-not-allowed hover:bg-gray-800"
          >
            Checkout
          </button>
        </aside>
      </div>
    </div>

    <!-- ============================================ -->
    <!-- FAVOURITES PAGE -->
    <!-- ============================================ -->
    <div
      v-else-if="currentPage === 'favourites'"
      class="flex-1 overflow-y-auto px-6 py-6 bg-gray-50"
    >
      <div class="flex items-center justify-between mb-4">
        <h1 class="text-2xl font-bold text-black">
          My Favourites ({{ favouriteCount }})
        </h1>
        <button
          class="px-3 py-1.5 text-sm border rounded-full hover:bg-gray-100 text-black border-gray-300"
          @click="currentPage = 'home'"
        >
          Back to Home
        </button>
      </div>

      <div v-if="favourites.length === 0" class="text-black">
        No favourite items yet
      </div>

      <div v-else class="flex flex-wrap gap-4">
        <div
          v-for="item in favourites"
          :key="item.name"
          class="bg-white rounded-2xl p-3 shadow-md w-48 sm:w-56 md:w-64 lg:w-72 relative"
        >
          <!-- Favourite Toggle Button -->
          <button class="absolute top-4 right-4" @click="toggleFavourite(item)">
            <Icon
              icon="fluent-emoji-high-contrast:star"
              width="30"
              height="30"
              :color="item.isFavourite ? 'orange' : 'white'"
            />
          </button>

          <!-- Product Image -->
          <img
            :src="item.img"
            alt=""
            class="w-full h-[180px] object-cover rounded-xl"
          />

          <!-- Product Details -->
          <div class="p-2">
            <h2 class="text-xs font-semibold text-black">{{ item.name }}</h2>
            <p class="text-gray-600 text-[10px] mt-1">{{ item.description }}</p>

            <!-- Price & Add to Cart -->
            <div class="mt-2 flex items-center justify-between gap-2">
              <span
                class="text-green-600 font-semibold text-xs whitespace-nowrap"
              >
                ฿{{ item.price }}
              </span>
              <button
                class="px-3 py-1.5 rounded-full bg-orange-500 text-white font-medium hover:bg-orange-600 text-xs whitespace-nowrap shrink-0"
                @click="addToCart(item, 1)"
              >
                Add to Cart
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Additional custom styles can be added here if needed */
</style>
