
(() => {
  "use strict";

  const PAGE = "vente-particulier";
  const KEY = "wl_auto_sales_v1";
  const SUCCESS = "la vente s'est effectuée avec succès";
  const DEFAULTS = {
    excluded: ["LBinx", "Papy rasta"],
    vip: ["dunn30", "BAK", "breizh-botanic"],
    normalPrice: 5,
    vipPrice: 3,
    quantity: 1000,
    variety: ""
  };

  if (!location.href.includes(PAGE)) {
    alert("Ouvre d'abord la page Vente particulier de Weed-Land.");
    return;
  }

  const norm = v => String(v ?? "").trim().toLowerCase();
  const read = () => {
    try { return JSON.parse(sessionStorage.getItem(KEY)); }
    catch (_) { return null; }
  };
  const save = () => sessionStorage.setItem(KEY, JSON.stringify(state));

  let state = read() || {
    config: DEFAULTS,
    players: [],
    index: 0,
    pending: false,
    auto: false,
    served: [],
    failed: ""
  };

  state.config = { ...DEFAULTS, ...(state.config || {}) };

  const find = selector => document.querySelector(selector);
  const set = (el, value) => {
    if (!el) return;
    el.value = value;
    el.dispatchEvent(new Event("input", { bubbles: true }));
    el.dispatchEvent(new Event("change", { bubbles: true }));
  };

  function successVisible() {
    return norm(document.body.innerText).includes(SUCCESS);
  }

  function current() {
    return state.players[state.index] || null;
  }

  function excluded(p) {
    return state.config.excluded.some(n => norm(n) === norm(p.u));
  }

  function vip(p) {
    return state.config.vip.some(n => norm(n) === norm(p.u));
  }

  function price(p) {
    return vip(p)
      ? Number(state.config.vipPrice)
      : Number(state.config.normalPrice);
  }

  function removePanel() {
    document.getElementById("wl-auto-sales")?.remove();
  }

  function button(text, action, color = "#387b38") {
    const b = document.createElement("button");
    b.textContent = text;
    b.style.cssText =
      `padding:9px;margin:4px 3px;border:0;border-radius:6px;` +
      `background:${color};color:white;font-weight:bold;`;
    b.onclick = action;
    return b;
  }

  function input(labelText, value, update, type = "text") {
    const label = document.createElement("label");
    label.style.cssText = "display:block;margin:8px 0;font-weight:bold";
    label.append(document.createTextNode(labelText));
    const el = document.createElement("input");
    el.type = type;
    el.value = value;
    el.style.cssText =
      "display:block;width:100%;box-sizing:border-box;padding:7px;margin-top:4px";
    el.onchange = () => { update(el.value); save(); };
    label.append(el);
    return label;
  }

  function prepare(p) {
    const buyer = find('input[name="acheteur"]');
    const variety = find('select[name="variete_vendu"]');
    const quantity = find('input[name="cq"]');
    const cost = find('input[name="prix"]');

    if (!buyer || !variety || !quantity || !cost) {
      alert("Formulaire introuvable : vérifie que tu es sur Vente particulier.");
      return false;
    }

    set(buyer, p.u);
    if (state.config.variety) set(variety, state.config.variety);
    set(quantity, state.config.quantity);
    set(cost, price(p));
    return true;
  }

  function processAfterReload() {
    if (!state.pending) return;

    if (successVisible()) {
      const p = current();
      if (p && !state.served.some(n => norm(n) === norm(p.u))) {
        state.served.push(p.u);
      }
      state.index++;
      state.pending = false;
      state.failed = "";
      save();
    } else {
      state.auto = false;
      state.failed =
        "Résultat incertain : aucun message de succès détecté. " +
        "Vérifie le tableau des ventes avant de reprendre.";
      save();
    }
  }

  function render() {
    removePanel();
    const panel = document.createElement("div");
    panel.id = "wl-auto-sales";
    panel.style.cssText =
      "position:fixed;z-index:2147483647;top:10px;right:8px;" +
      "width:min(360px,94vw);max-height:85vh;overflow:auto;" +
      "background:white;color:#222;border:2px solid #387b38;" +
      "border-radius:12px;padding:12px;box-shadow:0 3px 18px #0005;" +
      "font:14px Arial,sans-serif";

    const title = document.createElement("h3");
    title.textContent = "🌱 Weed-Land : ventes";
    panel.append(title, button("Fermer", removePanel, "#666"));

    panel.append(input("Exclusions, séparées par des virgules",
      state.config.excluded.join(", "),
      v => state.config.excluded = v.split(",").map(s => s.trim()).filter(Boolean)));

    panel.append(input("Joueurs VIP, séparés par des virgules",
      state.config.vip.join(", "),
      v => state.config.vip = v.split(",").map(s => s.trim()).filter(Boolean)));

    panel.append(input("Prix normal par gramme",
      state.config.normalPrice,
      v => state.config.normalPrice = Number(v) || 0, "number"));

    panel.append(input("Prix VIP par gramme",
      state.config.vipPrice,
      v => state.config.vipPrice = Number(v) || 0, "number"));

    panel.append(input("Quantité en grammes",
      state.config.quantity,
      v => state.config.quantity = Math.max(1, Number(v) || 1), "number"));

    const fileLabel = document.createElement("label");
    fileLabel.style.cssText = "display:block;margin:12px 0;font-weight:bold";
    fileLabel.append(document.createTextNode("Importer le JSON du jour "));
    const file = document.createElement("input");
    file.type = "file";
    file.accept = ".json,application/json";
    file.style.cssText = "display:block;width:100%;margin-top:5px";
    file.onchange = async () => {
      if (!file.files?.[0]) return;
      try {
        const json = JSON.parse(await file.files[0].text());
        const arr = Array.isArray(json) ? json :
          Array.isArray(json.players) ? json.players :
          Array.isArray(json.users) ? json.users : null;
        if (!arr) throw new Error("Le fichier doit contenir une liste de joueurs.");
        state.players = arr.filter(p => p?.u && !excluded(p))
          .map(p => ({ u: String(p.u).trim(), t: p.t ?? "", msg: p.msg ?? "" }));
        state.index = 0;
        state.pending = false;
        state.auto = false;
        state.served = [];
        state.failed = "";
        save();
        render();
        alert(state.players.length + " joueur(s) importé(s).");
      } catch (e) {
        alert("Import impossible : " + e.message);
      }
    };
    fileLabel.append(file);
    panel.append(fileLabel);

    const variety = find('select[name="variete_vendu"]');
    if (variety) {
      const label = document.createElement("label");
      label.textContent = "Variété à vendre";
      label.style.cssText = "display:block;margin:8px 0;font-weight:bold";
      const select = document.createElement("select");
      select.style.cssText = "display:block;width:100%;padding:7px";
      [...variety.options].forEach(opt => {
        const o = document.createElement("option");
        o.value = opt.value;
        o.textContent = opt.textContent;
        select.append(o);
      });
      select.value = state.config.variety || variety.value;
      select.onchange = () => {
        state.config.variety = select.value;
        save();
      };
      label.append(select);
      panel.append(label);
    }

    const p = current();
    const status = document.createElement("p");
    status.style.cssText = "white-space:pre-line;border-top:1px solid #ddd;padding-top:10px";
    status.textContent = state.failed
      ? "ARRÊT DE SÉCURITÉ\n" + state.failed
      : p
        ? `Joueur ${state.index + 1}/${state.players.length}\n` +
          `Acheteur : ${p.u}\nTarif : ${price(p)} WL/g` +
          (vip(p) ? " (VIP)" : "") +
          `\nQuantité : ${state.config.quantity} g\n` +
          `Total : ${price(p) * Number(state.config.quantity)} WL`
        : state.players.length
          ? "Tous les joueurs ont été traités."
          : "Importe d'abord le JSON du jour.";
    panel.append(status);

    if (p && !state.failed) {
      panel.append(button("Préparer cette vente", () => {
        if (prepare(p)) alert("Formulaire rempli. Vérifie les champs avant l'envoi.");
      }));

      panel.append(button("Marquer la vente comme envoyée", () => {
        if (!confirm("As-tu envoyé cette vente et vu le message de succès ?")) return;
        if (!successVisible()) {
          alert("Message de succès absent. Le joueur n'est pas marqué comme servi.");
          return;
        }
        state.served.push(p.u);
        state.index++;
        save();
        render();
      }, "#555"));

      panel.append(button("Joueur suivant (sans confirmer)", () => {
        if (!confirm("Passer ce joueur sans le marquer comme servi ?")) return;
        state.index++;
        save();
        render();
      }, "#996515"));
    }

    panel.append(button("Effacer la progression", () => {
      if (!confirm("Effacer la liste et la progression de la session ?")) return;
      sessionStorage.removeItem(KEY);
      location.reload();
    }, "#777"));

    document.body.append(panel);
  }

  processAfterReload();
  save();
  render();
})();
