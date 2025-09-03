<!-- Profil README.md - Assyifaul04 -->

<h1 align="center">🔥 Ngoding Dulu, Jagonya Belakangan!</h1>

import React, { useEffect, useState, useRef } from "react";

// SpotifyReminder.jsx
// React component (single-file) menggunakan TailwindCSS
// Fitur:
// - Menampilkan kartu "Spotify" sederhana untuk lagu Niken Salindri
// - Tombol Play (membuka Spotify search link)
// - Set reminder harian pada jam yang dipilih (disimpan di localStorage)
// - Notifikasi browser saat waktu reminder tiba (hanya saat halaman terbuka)

const SPOTIFY_SEARCH_URL =
  "https://open.spotify.com/search/Niken%20Salindri";
const STORAGE_KEY = "spotify_reminder_niken";

export default function SpotifyReminder({ coverUrl }) {
  const [enabled, setEnabled] = useState(false);
  const [time, setTime] = useState("08:00");
  const [message, setMessage] = useState("");
  const [permission, setPermission] = useState(Notification?.permission || "default");
  const triggeredRef = useRef(false); // mencegah notifikasi berulang dalam menit yang sama

  // load saved settings
  useEffect(() => {
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        const data = JSON.parse(raw);
        setEnabled(Boolean(data.enabled));
        if (data.time) setTime(data.time);
      }
    } catch (e) {
      console.warn("Failed to load reminder settings", e);
    }
  }, []);

  // save settings
  useEffect(() => {
    localStorage.setItem(
      STORAGE_KEY,
      JSON.stringify({ enabled: Boolean(enabled), time })
    );
  }, [enabled, time]);

  // request Notification permission
  const requestPermission = async () => {
    if (!("Notification" in window)) {
      setMessage("Browser tidak mendukung notifikasi.");
      return;
    }
    const p = await Notification.requestPermission();
    setPermission(p);
    if (p === "granted") setMessage("Notifikasi diizinkan ✅");
    else setMessage("Notifikasi tidak diizinkan.");
  };

  // check every 15s apakah waktu sudah tiba (hanya bekerja saat tab terbuka)
  useEffect(() => {
    const check = () => {
      if (!enabled) {
        triggeredRef.current = false;
        return;
      }
      const now = new Date();
      const [h, m] = time.split(":").map((s) => parseInt(s, 10));
      if (now.getHours() === h && now.getMinutes() === m) {
        if (!triggeredRef.current) {
          triggeredRef.current = true;
          sendReminder();
        }
      } else {
        triggeredRef.current = false;
      }
    };

    check();
    const id = setInterval(check, 15_000);
    return () => clearInterval(id);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [enabled, time]);

  const sendReminder = () => {
    // visual toast kecil
    setMessage("Waktunya dengerin: Niken Salindri 🎧");

    // notifikasi browser kalau diizinkan
    if (permission === "granted") {
      try {
        const n = new Notification("Spotify Reminder", {
          body: "Saatnya dengarkan Niken Salindri — klik untuk buka Spotify",
          tag: "spotify-reminder-niken",
          renotify: true,
        });
        n.onclick = () => {
          window.open(SPOTIFY_SEARCH_URL, "_blank");
          n.close();
        };
      } catch (e) {
        console.warn("Gagal membuat notifikasi", e);
      }
    }

    // fallback: buka spotify di tab baru (tidak otomatis tanpa interaksi pada banyak browser)
    // jadi kita tampilkan tombol Play pada kartu untuk user klik
  };

  const handleToggle = () => {
    setEnabled((s) => !s);
    // minta permission saat user baru mengaktifkan
    if (!enabled && Notification && Notification.permission === "default") {
      requestPermission();
    }
  };

  const defaultCover =
    coverUrl ||
    "https://via.placeholder.com/640x640.png?text=Niken+Salindri";

  return (
    <div className="max-w-xl mx-auto p-4">
      <div className="bg-white rounded-2xl shadow-lg overflow-hidden flex gap-4 p-4 items-center">
        <img
          src={defaultCover}
          alt="Niken Salindri cover"
          className="w-24 h-24 rounded-lg object-cover flex-shrink-0"
        />

        <div className="flex-1">
          <h3 className="text-lg font-semibold">Niken Salindri</h3>
          <p className="text-sm text-gray-500 mt-1">Spotify • Playlist / Single</p>

          <div className="mt-3 flex items-center gap-2">
            <a
              href={SPOTIFY_SEARCH_URL}
              target="_blank"
              rel="noreferrer"
              className="inline-flex items-center gap-2 px-3 py-2 rounded-xl font-medium shadow-md hover:shadow-lg transition"
              style={{ backgroundColor: "#1DB954", color: "white" }}
            >
              ▶️ Putar di Spotify
            </a>

            <button
              onClick={() => {
                // quick open without notification
                window.open(SPOTIFY_SEARCH_URL, "_blank");
              }}
              className="ml-2 text-sm px-3 py-2 rounded-xl border"
            >
              Buka
            </button>
          </div>

          <div className="mt-3 grid grid-cols-2 gap-2 items-center">
            <div>
              <label className="block text-xs text-gray-600">Pengingat Harian</label>
              <div className="mt-1 flex items-center gap-2">
                <input
                  type="time"
                  value={time}
                  onChange={(e) => setTime(e.target.value)}
                  className="border rounded px-2 py-1 text-sm"
                />
                <label className="inline-flex items-center gap-2 text-sm">
                  <input
                    type="checkbox"
                    checked={enabled}
                    onChange={handleToggle}
                    className="w-5 h-5"
                  />
                  Aktif
                </label>
              </div>
            </div>

            <div>
              <label className="block text-xs text-gray-600">Status Notifikasi</label>
              <div className="mt-1 text-sm">
                <div>{permission === "granted" ? "Diizinkan ✅" : permission === "denied" ? "Ditolak ❌" : "Belum diminta"}</div>
                <div className="mt-2">
                  <button
                    onClick={requestPermission}
                    className="text-xs px-2 py-1 rounded border"
                  >
                    Minta Izin Notifikasi
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      {message && (
        <div className="mt-3 p-3 bg-yellow-50 border-l-4 border-yellow-400 rounded">
          <div className="text-sm">{message}</div>
        </div>
      )}

      <div className="mt-4 text-xs text-gray-500">
        <div>Catatan:</div>
        <ul className="list-disc ml-5 mt-1">
          <li>Notifikasi hanya muncul saat halaman / tab terbuka.</li>
          <li>
            Untuk pengingat yang bekerja selalu (walau tab ditutup), gunakan integrasi Kalender
            / service worker / aplikasi mobile.
          </li>
          <li>Komponen ini menggunakan Tailwind CSS untuk styling.</li>
        </ul>
      </div>
    </div>
  );
}



