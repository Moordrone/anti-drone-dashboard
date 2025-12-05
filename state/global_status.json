const STATUS_URL = "state/global_status.json";
const REFRESH_INTERVAL_MS = 30000; // 30 s

async function fetchStatus() {
  try {
    const response = await fetch(STATUS_URL + "?_t=" + Date.now());
    if (!response.ok) {
      throw new Error("HTTP " + response.status);
    }
    const data = await response.json();
    updateDashboard(data);
  } catch (err) {
    console.error("Erreur de chargement du status :", err);
    setErrorState("Impossible de charger les données (" + err.message + ")");
  }
}

function updateDashboard(data) {
  // Exemple de structure attendue (à adapter si besoin) :
  // {
  //   "last_update": "2025-12-05T15:32:00Z",
  //   "alert": {"level": "HIGH", "description": "Drone FPV en approche"},
  //   "summary": {"total_threats": 3, "active_sensors": 5},
  //   "sensors": [
  //     {"name": "Radar X-band", "status": "OK", "details": "Portée 5 km"},
  //     ...
  //   ],
  //   "targets": [
  //     {"id": "T1", "type": "Drone FPV", "range_km": 2.3, "bearing_deg": 120, "status": "TRACKING"},
  //     ...
  //   ],
  //   "events": [
  //     {"time": "2025-12-05T15:31:00Z", "level": "WARN", "message": "Perte de lock radar momentanée"},
  //     ...
  //   ]
  // }

  // --- Dernière mise à jour ---
  const lastUpdateEl = document.getElementById("last-update-value");
  if (data.last_update) {
    const d = new Date(data.last_update);
    lastUpdateEl.textContent = d.toLocaleString("fr-FR");
  } else {
    lastUpdateEl.textContent = "–";
  }

  // --- Résumé ---
  document.getElementById("total-threats").textContent =
    data.summary?.total_threats ?? "0";

  document.getElementById("active-sensors").textContent =
    data.summary?.active_sensors ?? "0";

  // --- Alerte ---
  const alertCard = document.getElementById("alert-card");
  const alertLevelEl = document.getElementById("alert-level");
  const alertDescEl = document.getElementById("alert-description");

  alertCard.classList.remove("alert-low", "alert-medium", "alert-high");

  const level = (data.alert?.level || "UNKNOWN").toString().toUpperCase();
  alertLevelEl.textContent = level;

  if (data.alert?.description) {
    alertDescEl.textContent = data.alert.description;
  } else {
    alertDescEl.textContent = "Aucune alerte active.";
  }

  if (level === "LOW") {
    alertCard.classList.add("alert-low");
  } else if (level === "MEDIUM" || level === "MED") {
    alertCard.classList.add("alert-medium");
  } else if (level === "HIGH" || level === "CRITICAL") {
    alertCard.classList.add("alert-high");
  }

  // --- Capteurs ---
  const sensorList = document.getElementById("sensor-list");
  sensorList.innerHTML = "";

  if (Array.isArray(data.sensors) && data.sensors.length > 0) {
    data.sensors.forEach((s) => {
      const li = document.createElement("li");
      const status = (s.status || "").toString().toUpperCase();

      let badgeClass = "badge-ok";
      if (status === "WARN" || status === "DEGRADED") badgeClass = "badge-warn";
      if (status === "ERR" || status === "DOWN" || status === "OFFLINE")
        badgeClass = "badge-err";

      li.innerHTML = `
        <strong>${s.name || "Capteur"}</strong>
        <span class="badge ${badgeClass}">${status || "UNK"}</span>
        ${s.details ? `<div class="muted">${s.details}</div>` : ""}
      `;
      sensorList.appendChild(li);
    });
  } else {
    const li = document.createElement("li");
    li.className = "muted";
    li.textContent = "Aucun capteur dans les données.";
    sensorList.appendChild(li);
  }

  // --- Cibles ---
  const targetList = document.getElementById("target-list");
  targetList.innerHTML = "";

  if (Array.isArray(data.targets) && data.targets.length > 0) {
    data.targets.forEach((t) => {
      const li = document.createElement("li");
      li.innerHTML = `
        <strong>${t.id || "Cible"}</strong>
        ${t.type ? ` – ${t.type}` : ""}
        ${
          t.range_km != null
            ? `<div class="muted">Distance : ${t.range_km.toFixed?.(2) ?? t.range_km} km</div>`
            : ""
        }
        ${
          t.bearing_deg != null
            ? `<div class="muted">Azimut : ${t.bearing_deg}°</div>`
            : ""
        }
        ${t.status ? `<div class="muted">État : ${t.status}</div>` : ""}
      `;
      targetList.appendChild(li);
    });
  } else {
    const li = document.createElement("li");
    li.className = "muted";
    li.textContent = "Aucune cible active.";
    targetList.appendChild(li);
  }

  // --- Journal ---
  const logList = document.getElementById("event-log");
  logList.innerHTML = "";

  if (Array.isArray(data.events) && data.events.length > 0) {
    data.events.forEach((e) => {
      const li = document.createElement("li");

      const t = e.time ? new Date(e.time).toLocaleTimeString("fr-FR") : "";
      const lvl = (e.level || "INFO").toString().toUpperCase();

      let lvlClass = "log-level-info";
      if (lvl === "WARN") lvlClass = "log-level-warn";
      if (lvl === "ERR" || lvl === "ERROR" || lvl === "ALERT")
        lvlClass = "log-level-err";

      li.innerHTML = `
        ${t ? `<span class="log-time">${t}</span>` : ""}
        <span class="log-level ${lvlClass}">[${lvl}]</span>
        <span>${e.message || ""}</span>
      `;
      logList.appendChild(li);
    });
  } else {
    const li = document.createElement("li");
    li.className = "muted";
    li.textContent = "Aucun événement dans le journal.";
    logList.appendChild(li);
  }
}

function setErrorState(msg) {
  document.getElementById("alert-description").textContent = msg;
}

document.addEventListener("DOMContentLoaded", () => {
  fetchStatus();
  setInterval(fetchStatus, REFRESH_INTERVAL_MS);
});
