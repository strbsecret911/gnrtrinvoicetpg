/* =========================================================
   Firebase CDN (v9 Modular) - GitHub Pages Friendly
========================================================= */
import { initializeApp } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js";
import { getAnalytics } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-analytics.js";
import {
  getFirestore,
  doc,
  getDoc,
  setDoc,
  runTransaction,
  serverTimestamp
} from "https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js";

/* =========================================================
   Firebase Config (PUNYAMU)
========================================================= */
const firebaseConfig = {
  apiKey: "AIzaSyC-wCxDBYk8ANOOpRgtxQWc9Vf7hmWnAEg",
  authDomain: "invoicetpggenerator.firebaseapp.com",
  projectId: "invoicetpggenerator",
  storageBucket: "invoicetpggenerator.firebasestorage.app",
  messagingSenderId: "727612866580",
  appId: "1:727612866580:web:883767eab2691d2c802701",
  measurementId: "G-YJVGC0Q4X4"
};

const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);
const db = getFirestore(app);

/* =========================================================
   KONFIGURASI APLIKASI
========================================================= */
const PREFIX = "TPG";
const FIXED = "0425"; // format kode: TPG+0425+KODE+7DIGIT

const CATALOG = {
  "Paket Internet": [
    { label: "Internet Umum", code: "INET" },
    { label: "Telkomsel", code: "INETTS" },
    { label: "XL", code: "INETXL" },
    { label: "Indosat", code: "INETISAT" }
  ],
  "Topup Games": [
    { label: "Mobile Legends", code: "ML" },
    { label: "Free Fire", code: "FF" },
    { label: "Love and Deepspace", code: "LADS" },
    { label: "Genshin Impact", code: "GI" },
    { label: "Heartopia", code: "HRT" },
    { label: "Roblox", code: "RBX" },
    { label: "Honkai Star Rail", code: "HSR" },
    { label: "Honor of Kings", code: "HOK" },
    { label: "DLL / Custom", code: "DLL" }
  ]
};

/* =========================================================
   HELPER
========================================================= */
const $ = (id) => document.getElementById(id);

function pad(n, l) {
  return String(n).padStart(l, "0");
}

function rupiah(n) {
  return "Rp " + Number(n || 0).toLocaleString("id-ID");
}

function random7Digit() {
  return pad(Math.floor(Math.random() * 10_000_000), 7);
}

function buildCode(productCode) {
  return `${PREFIX}+${FIXED}+${productCode}+${random7Digit()}`;
}

function status(text, type = "info") {
  const el = $("status");
  el.textContent = text;
  el.style.color =
    type === "success" ? "#22c55e" :
    type === "error" ? "#ef4444" :
    "#a8b0c7";
}

/* =========================================================
   FIRESTORE LOGIC
========================================================= */

// Generate invoice global (counter)
async function generateInvoice() {
  const ref = doc(db, "counters", "invoice");

  const number = await runTransaction(db, async (tx) => {
    const snap = await tx.get(ref);
    const current = snap.exists() ? snap.data().current || 0 : 0;
    const next = current + 1;
    tx.set(ref, { current: next }, { merge: true });
    return next;
  });

  const d = new Date();
  return `INV-${d.getFullYear()}${pad(d.getMonth()+1,2)}${pad(d.getDate(),2)}-${pad(number,6)}`;
}

// Generate kode pembelian (sekali pakai)
async function generateUniqueCode(productCode) {
  for (let i = 0; i < 20; i++) {
    const code = buildCode(productCode);
    const ref = doc(db, "codes", code);
    const snap = await getDoc(ref);

    if (!snap.exists()) {
      await setDoc(ref, {
        productCode,
        createdAt: serverTimestamp()
      });
      return code;
    }
  }
  throw new Error("Gagal membuat kode unik");
}

// Simpan struk
async function saveReceipt(data) {
  await setDoc(doc(db, "receipts", data.invoice), {
    ...data,
    createdAt: serverTimestamp()
  });
}

/* =========================================================
   UI LOGIC
========================================================= */
function initCategory() {
  $("category").innerHTML = "";
  Object.keys(CATALOG).forEach(cat => {
    const o = document.createElement("option");
    o.value = cat;
    o.textContent = cat;
    $("category").appendChild(o);
  });
  initProduct();
}

function initProduct() {
  const cat = $("category").value;
  $("product").innerHTML = "";
  CATALOG[cat].forEach(p => {
    const o = document.createElement("option");
    o.value = p.code;
    o.textContent = p.label;
    $("product").appendChild(o);
  });
}

function updatePreview(invoice = "-", code = "-") {
  const qty = Number($("qty").value || 0);
  const price = Number($("price").value || 0);

  $("rInvoice").textContent = invoice;
  $("rCode").textContent = code;
  $("rBuyer").textContent = $("buyerName").value || "-";
  $("rBuyerId").textContent = $("buyerId").value || "-";
  $("rCategory").textContent = $("category").value;
  $("rProduct").textContent =
    $("product").selectedOptions[0].text;
  $("rQty").textContent = qty;
  $("rPrice").textContent = rupiah(price);
  $("rTotal").textContent = rupiah(qty * price);
  $("rNote").textContent = $("note").value || "-";
}

/* =========================================================
   ACTIONS
========================================================= */
$("category").onchange = initProduct;

$("btnGenerate").onclick = async () => {
  try {
    status("Generate invoice & kode...");
    $("btnGenerate").disabled = true;

    const invoice = await generateInvoice();
    const code = await generateUniqueCode($("product").value);

    updatePreview(invoice, code);

    await saveReceipt({
      invoice,
      code,
      buyer: $("buyerName").value,
      buyerId: $("buyerId").value,
      category: $("category").value,
      product: $("product").value,
      qty: Number($("qty").value),
      price: Number($("price").value),
      total: Number($("qty").value) * Number($("price").value),
      note: $("note").value
    });

    status("Berhasil disimpan", "success");
  } catch (e) {
    console.error(e);
    status(e.message, "error");
  } finally {
    $("btnGenerate").disabled = false;
  }
};

$("btnSaveJpg").onclick = async () => {
  const canvas = await html2canvas($("receipt"), {
    scale: 2,
    backgroundColor: suggests: "#0b1226"
  });
  const a = document.createElement("a");
  a.href = canvas.toDataURL("image/jpeg", 0.95);
  a.download = `TOPUPGRAM-${$("rInvoice").textContent}.jpg`;
  a.click();
};

/* =========================================================
   INIT
========================================================= */
initCategory();
updatePreview();
