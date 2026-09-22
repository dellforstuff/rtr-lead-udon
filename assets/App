/* ------------------------------------------------------------------
   RTR Lead — สคริปต์กลาง ใช้ร่วมกันทุกหน้า
   แก้ค่า API ด้านล่างเพียงจุดเดียว ทุกหน้าจะเปลี่ยนตามทั้งหมด
------------------------------------------------------------------ */

var API = "https://script.google.com/macros/s/AKfycbwlCFXc0PRBndFky2UWw5vGW5vtFy_mwDUeI9ef0uLQUV-nKVDRjfR2rZiSpGl4fRPdGw/exec";

/* รายการพื้นที่โหลดจากแท็บ Areas ตอนเปิดหน้า
   เพิ่มจังหวัด/อำเภอในชีตแล้วหน้าเว็บอัปเดตเอง ไม่ต้องแก้โค้ด
   ชุดด้านล่างเป็นตัวสำรองไว้ใช้เมื่อโหลดจากเซิร์ฟเวอร์ไม่ได้ */
var AREAS_FALLBACK = [];   // ระบบอุดรจ่ายลีดผ่าน CM ของร้าน ไม่ได้ใช้พื้นที่ในการจ่ายงาน
                           // รายการพื้นที่จริงโหลดจากแท็บ Areas ตอนเปิดหน้า
var AREAS = AREAS_FALLBACK;

var PACKAGES = [
  "แพ็กเน็ตบ้านราคาประหยัด 499",
  "แพ็กผู้ประกอบการ 699",
  "สนใจสอบถามแพ็กเกจอื่น"
];

/* รูปโปรโมชั่นที่แสดงเหนือช่องเลือกแพ็กเกจ
   ไฟล์ต้องอยู่ที่ assets/promo.jpg ใน repo เดียวกัน
   ตั้งเป็นค่าว่างถ้าไม่ต้องการแสดงรูป */
var PROMO_IMAGE = "assets/promo.jpg";

var CALL_TIMES = [
  { v: "เช้า (08:00-12:00)", a: "เช้า", b: "08:00-12:00" },
  { v: "บ่าย (12:00-17:00)", a: "บ่าย", b: "12:00-17:00" },
  { v: "เย็น (17:00-21:00)", a: "เย็น", b: "17:00-21:00" },
  { v: "เวลาไหนก็ได้", a: "เวลาไหน", b: "ก็ได้" }
];

function $(id) { return document.getElementById(id); }

/* หา base path ของเว็บ รองรับทั้ง custom domain และ username.github.io/reponame */
function baseUrl() {
  if (typeof window.RTR_BASE === "string") return window.RTR_BASE;
  var seg = location.pathname.split("/").filter(Boolean);
  if (/\.github\.io$/i.test(location.hostname) && seg.length && seg[0] !== "r") {
    return "/" + seg[0] + "/";
  }
  return "/";
}

function readSlug() {
  if (window.RTR_SLUG) return String(window.RTR_SLUG).toUpperCase();
  var q = new URLSearchParams(location.search).get("s");
  if (q) return q.trim().toUpperCase();
  var parts = location.pathname.split("/").filter(Boolean);
  var last = parts[parts.length - 1] || "";
  return /^[A-Z0-9]{4}$/i.test(last) ? last.toUpperCase() : "";
}

function opts(list) {
  return list.map(function (v) {
    return '<option value="' + v + '">' + v + "</option>";
  }).join("");
}

/** รูปโปรโมชั่น กดแล้วเปิดดูขนาดเต็มได้ เผื่อตัวหนังสือในรูปเล็กเกินไปบนมือถือ */
function promoHtml() {
  if (!PROMO_IMAGE) return "";
  var src = baseUrl() + PROMO_IMAGE;
  // แสดงเต็มภาพในหน้าเลย ไม่ต้องกดเปิดแท็บใหม่
  // ยังกดได้อยู่เผื่อลูกค้าอยากซูมดูตัวหนังสือเล็ก ๆ
  return '<a class="promo" href="' + src + '" target="_blank" rel="noopener">' +
    '<img src="' + src + '" alt="โปรโมชั่นแพ็กเกจเน็ตบ้าน"></a>';
}

function areaOpts() {
  return AREAS.map(function (a) {
    return '<option value="' + a.code + '">' + a.label + "</option>";
  }).join("") + '<option value="OTHER">พื้นที่อื่น (ไม่อยู่ในรายการ)</option>';
}

/** เติมรายการพื้นที่ลง dropdown โดยไม่ทำให้ตัวเลือกที่ผู้ใช้เลือกไว้หาย */
function fillAreaSelect(preferCode) {
  var sel = $("district");
  if (!sel) return;
  var keep = sel.value;
  sel.innerHTML = '<option value="">— เลือกพื้นที่ติดตั้ง —</option>' + areaOpts();
  var want = keep || preferCode || "";
  if (want) {
    for (var i = 0; i < sel.options.length; i++) {
      if (sel.options[i].value === want) { sel.selectedIndex = i; break; }
    }
  }
}

function chips() {
  return CALL_TIMES.map(function (c) {
    return '<label class="chip"><input type="radio" name="preferred_call_time" value="' +
      c.v + '"><span>' + c.a + "<br>" + c.b + "</span></label>";
  }).join("");
}

/* ------------------------------------------------------------------ */

function renderForm(mountId) {
  $(mountId).innerHTML = [
    '<form class="card" id="form" novalidate>',
    '<label for="name">ชื่อ-นามสกุล <span class="req">*</span></label>',
    '<input id="name" autocomplete="name" placeholder="เช่น สมหญิง ใจดี">',
    '<div class="err">กรุณากรอกชื่อ-นามสกุล</div>',

    '<label for="phone">เบอร์โทรศัพท์ <span class="req">*</span></label>',
    '<input id="phone" type="tel" inputmode="numeric" maxlength="10" autocomplete="tel" placeholder="0812345678">',
    '<div class="hint">เจ้าหน้าที่จะโทรกลับตามเบอร์นี้ กรุณากรอกให้ถูกต้อง</div>',
    '<div class="err">กรุณากรอกเบอร์มือถือ 10 หลัก ขึ้นต้นด้วย 0</div>',

    '<label for="district">พื้นที่ที่จะติดตั้ง <span class="req">*</span></label>',
    '<select id="district"><option value="">— เลือกพื้นที่ติดตั้ง —</option>' + areaOpts() + "</select>",
    '<div class="hint">เลือกอำเภอและจังหวัดที่จะติดตั้งจริง ไม่ใช่ที่ตั้งร้าน</div>',
    '<div class="err">กรุณาเลือกพื้นที่ติดตั้ง</div>',

    '<label for="addr">ที่อยู่สำหรับติดตั้ง <span class="req">*</span></label>',
    '<textarea id="addr" placeholder="บ้านเลขที่ / หมู่ / ซอย / ถนน / ตำบล"></textarea>',
    '<div class="err">กรุณากรอกที่อยู่ติดตั้ง</div>',

    '<label for="pkg">แพ็กเกจที่สนใจ <span class="req">*</span></label>',
    promoHtml(),
    '<select id="pkg"><option value="">— เลือกแพ็กเกจ —</option>' + opts(PACKAGES) + "</select>",
    '<div class="err">กรุณาเลือกแพ็กเกจ</div>',

    '<label>ช่วงเวลาที่สะดวกรับสาย <span class="req">*</span></label>',
    '<div class="chips" id="chips">' + chips() + "</div>",
    '<div class="err" id="timeErr">กรุณาเลือกช่วงเวลาที่สะดวก</div>',

    '<label for="note">รายละเอียดเพิ่มเติม</label>',
    '<textarea id="note" placeholder="เช่น อยู่คอนโด ชั้น 8 (ไม่บังคับ)"></textarea>',

    '<div class="hp"><label for="company">Company</label>',
    '<input id="company" tabindex="-1" autocomplete="off"></div>',

    '<div class="consent"><input type="checkbox" id="consent">',
    '<span class="t">ข้าพเจ้ายินยอมให้จัดเก็บและใช้ข้อมูลข้างต้นเพื่อติดต่อกลับและดำเนินการติดตั้งบริการ ',
    'และรับทราบว่าสามารถขอแก้ไขหรือลบข้อมูลได้ตลอดเวลา</span></div>',
    '<div class="err" id="consentErr">กรุณายอมรับเงื่อนไขก่อนส่งข้อมูล</div>',

    '<button type="submit" id="submitBtn">ส่งข้อมูล</button>',
    '<div class="foot">ข้อมูลของท่านถูกส่งผ่านการเชื่อมต่อที่เข้ารหัส</div>',
    "</form>"
  ].join("");
}

/** แปลง area_code เป็นข้อความอ่านออก ใช้ตอนส่งข้อมูลและตอนแสดงผล */
function areaLabel(code) {
  if (!code || code === "OTHER") return "พื้นที่อื่น";
  for (var i = 0; i < AREAS.length; i++) {
    if (AREAS[i].code === code) return AREAS[i].label;
  }
  return code;
}

function markBad(el, bad) {
  var e = el;
  while (e && !(e.classList && e.classList.contains("err"))) e = e.nextElementSibling;
  if (bad) { el.parentElement.classList.add("bad"); if (e) e.style.display = "block"; }
  else { el.parentElement.classList.remove("bad"); if (e) e.style.display = "none"; }
}

function setBanner(msg) {
  var b = $("banner");
  if (!b) return;
  b.textContent = msg;
  b.style.display = "block";
}

/* ------------------------------------------------------------------ */

function initLanding() {
  var SLUG = readSlug();
  var startedAt = Date.now();

  renderForm("formMount");

  // ---- โหลดรายการพื้นที่จากชีต
  if (API.indexOf("PASTE_") !== 0) {
    fetch(API + "?action=areas")
      .then(function (r) { return r.json(); })
      .then(function (d) {
        if (d && d.ok && d.areas && d.areas.length) {
          AREAS = d.areas;
          fillAreaSelect();
        }
      })
      .catch(function () { /* ใช้รายการสำรองต่อไป ผู้ใช้ยังกรอกได้ */ });
  }

  // ---- แสดงชื่อร้าน
  var nameEl = $("shopName");
  if (!SLUG) {
    if (nameEl) nameEl.textContent = "ไม่พบรหัสร้าน";
    setBanner("ลิงก์นี้ไม่มีรหัสร้านกำกับ ท่านยังส่งข้อมูลได้ตามปกติ แต่ระบบจะไม่สามารถระบุร้านที่แนะนำได้");
  } else if (nameEl) {
    nameEl.textContent = "รหัส " + SLUG;
    if (API.indexOf("PASTE_") !== 0) {
      fetch(API + "?action=shop&slug=" + encodeURIComponent(SLUG))
        .then(function (r) { return r.json(); })
        .then(function (d) {
          if (d && d.ok && d.shop_name) {
            nameEl.textContent = d.shop_name;
            if (d.district) {
              var sel = $("district");
              for (var i = 0; i < sel.options.length; i++) {
                if (sel.options[i].value === d.district) { sel.selectedIndex = i; break; }
              }
            }
          } else {
            setBanner("ไม่พบร้านที่ตรงกับรหัสนี้ในระบบ ท่านยังส่งข้อมูลได้ตามปกติ");
          }
        })
        .catch(function () { /* เงียบไว้ ให้ผู้ใช้กรอกต่อได้ */ });
    }
  }

  // ---- เบอร์โทร รับเฉพาะตัวเลข
  $("phone").addEventListener("input", function () {
    this.value = this.value.replace(/\D/g, "").slice(0, 10);
    if (/^0\d{9}$/.test(this.value)) markBad(this, false);
  });
  ["name", "district", "addr", "pkg"].forEach(function (id) {
    var el = $(id);
    ["input", "change"].forEach(function (ev) {
      el.addEventListener(ev, function () { if (this.value.trim()) markBad(this, false); });
    });
  });
  $("consent").addEventListener("change", function () {
    $("consentErr").style.display = this.checked ? "none" : "block";
  });
  Array.prototype.forEach.call(
    document.querySelectorAll('input[name="preferred_call_time"]'),
    function (r) { r.addEventListener("change", function () { $("timeErr").style.display = "none"; }); }
  );

  // ---- ส่งฟอร์ม
  $("form").addEventListener("submit", function (ev) {
    ev.preventDefault();
    var ok = true, firstBad = null;

    function need(id, valid) {
      var el = $(id), good = valid(el.value.trim());
      markBad(el, !good);
      if (!good) { ok = false; firstBad = firstBad || el; }
    }
    need("name", function (v) { return v.length >= 2; });
    need("phone", function (v) { return /^0\d{9}$/.test(v); });
    need("district", function (v) { return v !== ""; });
    need("addr", function (v) { return v.length >= 5; });
    need("pkg", function (v) { return v !== ""; });

    var t = document.querySelector('input[name="preferred_call_time"]:checked');
    $("timeErr").style.display = t ? "none" : "block";
    if (!t) { ok = false; firstBad = firstBad || $("chips"); }

    if (!$("consent").checked) {
      $("consentErr").style.display = "block";
      ok = false; firstBad = firstBad || $("consent");
    }
    if (!ok) { firstBad.scrollIntoView({ behavior: "smooth", block: "center" }); return; }

    if (API.indexOf("PASTE_") === 0) {
      setBanner("ยังไม่ได้ตั้งค่า API — เปิดไฟล์ assets/app.js แล้ววาง URL ของ Apps Script");
      return;
    }

    var btn = $("submitBtn");
    btn.disabled = true;
    btn.textContent = "กำลังส่ง…";

    fetch(API, {
      method: "POST",
      body: JSON.stringify({
        action: "create_lead",
        qr_slug: SLUG,
        customer_name: $("name").value.trim(),
        phone: $("phone").value.trim(),
        area_code: $("district").value === "OTHER" ? "" : $("district").value,
        district: areaLabel($("district").value),
        install_address: $("addr").value.trim(),
        package: $("pkg").value,
        preferred_call_time: t.value,
        customer_note: $("note").value.trim(),
        hp: $("company").value,
        elapsed_ms: Date.now() - startedAt,
        page_ref: location.href
      })
    })
      .then(function (r) { return r.json(); })
      .then(function (d) {
        if (!d || !d.ok) throw new Error("failed");
        $("formMount").classList.add("hide");
        var box = $("shopBox"); if (box) box.classList.add("hide");
        var ban = $("banner"); if (ban) ban.style.display = "none";
        $("refNo").textContent = "เลขอ้างอิง " + d.lead_id;
        $("done").classList.remove("hide");
        window.scrollTo({ top: 0, behavior: "smooth" });
      })
      .catch(function () {
        btn.disabled = false;
        btn.textContent = "ส่งข้อมูล";
        setBanner("ส่งข้อมูลไม่สำเร็จ กรุณาตรวจสอบสัญญาณอินเทอร์เน็ตแล้วลองใหม่อีกครั้ง");
        window.scrollTo({ top: 0, behavior: "smooth" });
      });
  });
}

/* ---- หน้ากรอกรหัสร้านเอง ใช้ตอน QR เลอะหรือสติกเกอร์ลอก ---- */
function initCodeEntry(inputId, btnId, errId) {
  var input = $(inputId);
  input.addEventListener("input", function () {
    this.value = this.value.replace(/[^A-Za-z0-9]/g, "").toUpperCase().slice(0, 4);
    $(errId).style.display = "none";
  });
  $(btnId).addEventListener("click", function () {
    var v = input.value.trim().toUpperCase();
    if (!/^[A-Z0-9]{4}$/.test(v)) { $(errId).style.display = "block"; return; }
    location.href = baseUrl() + "r/" + v + "/";
  });
}


/* ------------------------------------------------------------------
   initAuto — ใช้กับหน้าที่เป็นได้ทั้งฟอร์มและช่องกรอกรหัส
   ถ้าอ่านรหัสร้านจาก URL ได้ -> แสดงฟอร์ม
   ถ้าอ่านไม่ได้            -> แสดงช่องให้กรอกรหัสเอง
------------------------------------------------------------------ */
function initAuto() {
  if (readSlug()) {
    $("shopView").classList.remove("hide");
    initLanding();
  } else {
    $("codeView").classList.remove("hide");
    initCodeEntry("code", "go", "codeErr");
  }
}
