import React, { useEffect, useRef, useState } from "react";

// Simulador de Cultivo - Prototipo (single-file React component)
// TailwindCSS assumed available in the project.
// Exporta un componente por defecto que puede incrustarse en una página.

export default function VitroSimulador() {
  // Variables modificables
  const [temperature, setTemperature] = useState(24); // °C
  const [humidity, setHumidity] = useState(85); // %
  const [light, setLight] = useState(12); // horas
  const [sterility, setSterility] = useState(90); // % (práctica correcta)
  const [strain, setStrain] = useState("Pleurotus ostreatus");

  // Simulación
  const [time, setTime] = useState(0); // días simulados
  const [running, setRunning] = useState(false);
  const [growth, setGrowth] = useState<number[]>([0]); // crecimiento relativo 0-100
  const [events, setEvents] = useState<string[]>([]);
  const canvasRef = useRef<HTMLCanvasElement | null>(null);
  const rafRef = useRef<number | null>(null);

  // Score simple
  const score = Math.max(0, Math.round((temperatureScore() + humidityScore() + lightScore() + sterilityScore()) / 4));

  function temperatureScore() {
    // Ideal para muchos hongos 20-26°C -> 100
    const ideal = 24;
    const diff = Math.abs(temperature - ideal);
    return Math.max(0, 100 - diff * 8);
  }
  function humidityScore() {
    const ideal = 90;
    const diff = Math.abs(humidity - ideal);
    return Math.max(0, 100 - diff * 1.5);
  }
  function lightScore() {
    // Many fungi prefer low light but some light helps fructification
    const ideal = 12;
    const diff = Math.abs(light - ideal);
    return Math.max(0, 100 - diff * 6);
  }
  function sterilityScore() {
    return Math.max(0, Math.min(100, sterility));
  }

  // Simulación diaria (avance de crecimiento)
  function simulateDay(dayIndex: number) {
    // Baseline growth potential derived from scores
    const pot = (temperatureScore() * 0.25 + humidityScore() * 0.25 + lightScore() * 0.15 + sterilityScore() * 0.35) / 100;
    // growth delta randomizado en torno a la potencia
    const baseDelta = pot * (5 + Math.random() * 6); // 0-~10
    // Riesgo de contaminación si esterilidad baja
    const contaminationRisk = Math.max(0, 30 - sterility / 3); // 0-30
    const contamRoll = Math.random() * 100;

    let eventMsg = "";
    if (contamRoll < contaminationRisk) {
      // contaminación
      eventMsg = `Día ${dayIndex}: Contaminación detectada (riesgo ${Math.round(contaminationRisk)}%).`;
    } else if (Math.random() < 0.03) {
      eventMsg = `Día ${dayIndex}: Evento aleatorio (microfluctuación en humedad).`;
    }

    // Si hay contaminación, el crecimiento se frena
    const delta = eventMsg.includes("Contaminación") ? baseDelta * 0.25 : baseDelta;

    const prev = growth[growth.length - 1] ?? 0;
    let next = Math.min(100, prev + delta);
    return { next, eventMsg };
  }

  // Ejecutar simulación por días (no en tiempo real — se avanza por ticks)
  useEffect(() => {
    if (!running) return;
    let day = time;
    function step() {
      // avanzar un "día" cada 800ms (prototipo)
      day += 1;
      const { next, eventMsg } = simulateDay(day);
      setTime(day);
      setGrowth((g) => [...g, Math.round(next * 100) / 100]);
      if (eventMsg) setEvents((e) => [eventMsg, ...e].slice(0, 6));
      if (next >= 100) {
        setRunning(false);
      } else {
        rafRef.current = window.setTimeout(step, 800);
      }
    }
    rafRef.current = window.setTimeout(step, 800);
    return () => {
      if (rafRef.current) window.clearTimeout(rafRef.current);
    };
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [running]);

  // Canvas: dibuja micelio/colonización simple
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    if (!ctx) return;
    const w = canvas.width = canvas.clientWidth * window.devicePixelRatio;
    const h = canvas.height = canvas.clientHeight * window.devicePixelRatio;

    // limpiar
    ctx.clearRect(0, 0, w, h);

    // Background
    ctx.fillStyle = "#0f172a"; // dark
    ctx.fillRect(0, 0, w, h);

    // Dibuja placa/bolsa
    const pad = 20 * window.devicePixelRatio;
    const rectW = w - pad * 2;
    const rectH = h - pad * 2;
    ctx.fillStyle = "#071133";
    ctx.fillRect(pad, pad, rectW, rectH);

    // Growth visualization: usa growth last value
    const gval = (growth[growth.length - 1] ?? 0) / 100; // 0-1

    // dibuja "nube" de micelio con partículas
    const maxParticles = 160;
    const filled = Math.floor(maxParticles * gval);
    for (let i = 0; i < filled; i++) {
      const rx = pad + Math.random() * rectW;
      const ry = pad + Math.random() * rectH;
      const size = (1 + Math.random() * 4) * window.devicePixelRatio * (0.5 + gval);
      ctx.beginPath();
      ctx.fillStyle = `rgba(220, 235, 200, ${0.03 + 0.5 * gval})`;
      ctx.arc(rx, ry, size, 0, Math.PI * 2);
      ctx.fill();
    }

    // Overlay text
    ctx.fillStyle = "#e6eef8";
    ctx.font = `${14 * window.devicePixelRatio}px Inter, Roboto, sans-serif`;
    ctx.fillText(`Cepas: ${strain}`, pad + 6, pad + 18 * window.devicePixelRatio);
    ctx.fillText(`Crecimiento: ${Math.round((growth[growth.length - 1] ?? 0) * 100) / 100}%`, pad + 6, pad + 34 * window.devicePixelRatio);

  }, [growth, strain]);

  function resetSim() {
    setTime(0);
    setGrowth([0]);
    setEvents([]);
    setRunning(false);
  }

  // Simple SVG growth chart
  function GrowthChart({ data }: { data: number[] }) {
    const width = 300;
    const height = 120;
    if (data.length <= 1) {
      return (
        <div className="flex items-center justify-center text-sm text-zinc-400">Aún no hay datos.</div>
      );
    }
    const max = 100;
    const stepX = width / (data.length - 1);
    const points = data.map((d, i) => `${i * stepX},${height - (d / max) * height}`).join(" ");
    return (
      <svg viewBox={`0 0 ${width} ${height}`} className="w-full h-auto">
        <polyline
          fill="none"
          stroke="#60a5fa"
          strokeWidth={2}
          points={points}
        />
        {data.map((d, i) => (
          <circle key={i} cx={i * stepX} cy={height - (d / max) * height} r={2} fill="#60a5fa" />
        ))}
      </svg>
    );
  }

  return (
    <div className="max-w-5xl mx-auto p-4">
      <header className="flex items-center justify-between mb-4">
        <h1 className="text-2xl font-semibold">Simulador de Cultivo — VitroCorp</h1>
        <div className="text-sm text-zinc-500">Prototipo interactivo</div>
      </header>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {/* Panel controles */}
        <section className="col-span-1 md:col-span-1 bg-white/5 p-4 rounded-2xl shadow-sm">
          <h2 className="font-medium mb-3">Controles</h2>

          <label className="block text-xs text-zinc-400">Cepas</label>
          <select value={strain} onChange={(e) => setStrain(e.target.value)} className="w-full my-2 p-2 rounded-md bg-zinc-900">
            <option>Pleurotus ostreatus</option>
            <option>Hericium erinaceus</option>
            <option>Psilocybe cubensis (ej.)</option>
            <option>Planta (Albahaca)</option>
          </select>

          <label className="block text-xs text-zinc-400 mt-3">Temperatura: {temperature}°C</label>
          <input type="range" min={15} max={30} value={temperature} onChange={(e) => setTemperature(Number(e.target.value))} className="w-full" />

          <label className="block text-xs text-zinc-400 mt-3">Humedad: {humidity}%</label>
          <input type="range" min={50} max={100} value={humidity} onChange={(e) => setHumidity(Number(e.target.value))} className="w-full" />

          <label className="block text-xs text-zinc-400 mt-3">Luz (hrs/día): {light}h</label>
          <input type="range" min={0} max={24} value={light} onChange={(e) => setLight(Number(e.target.value))} className="w-full" />

          <label className="block text-xs text-zinc-400 mt-3">Esterilidad (prácticas %): {sterility}%</label>
          <input type="range" min={40} max={100} value={sterility} onChange={(e) => setSterility(Number(e.target.value))} className="w-full" />

          <div className="flex gap-2 mt-4">
            <button onClick={() => setRunning(true)} className="flex-1 py-2 bg-emerald-600 rounded-md">Iniciar</button>
            <button onClick={() => setRunning(false)} className="flex-1 py-2 bg-amber-500 rounded-md">Pausar</button>
            <button onClick={resetSim} className="flex-1 py-2 bg-zinc-700 rounded-md">Reset</button>
          </div>

          <div className="mt-4 text-sm text-zinc-300">
            <div>Puntuación: <span className="font-semibold">{score}/100</span></div>
            <div className="text-xs text-zinc-400">(Basada en temperatura, humedad, luz y esterilidad)</div>
          </div>
        </section>

        {/* Canvas y visual */}
        <section className="col-span-1 md:col-span-2 bg-white/5 p-4 rounded-2xl shadow-sm flex flex-col">
          <div className="flex items-start justify-between mb-3">
            <h2 className="font-medium">Visualización</h2>
            <div className="text-sm text-zinc-400">Día simulado: {time}</div>
          </div>

          <div className="flex gap-4">
            <div className="flex-1 bg-zinc-900 rounded-lg p-2">
              <div className="aspect-[16/9] w-full rounded-md overflow-hidden" style={{ minHeight: 220 }}>
                <canvas ref={canvasRef} style={{ width: '100%', height: '100%' }} />
              </div>
            </div>

            <aside className="w-72 bg-zinc-900 rounded-lg p-3 flex flex-col gap-3">
              <div>
                <h3 className="text-sm font-medium">Últimos eventos</h3>
                <ul className="text-xs text-zinc-400 mt-2 space-y-1">
                  {events.length === 0 ? <li>No hay eventos recientes.</li> : events.map((ev, i) => <li key={i}>• {ev}</li>)}
                </ul>
              </div>

              <div>
                <h3 className="text-sm font-medium">Resumen</h3>
                <div className="text-xs text-zinc-400">Temperatura opt: 20-26°C. Esterilidad alta reduce riesgo de contaminación.</div>
              </div>

              <div>
                <h3 className="text-sm font-medium">Consejos rápidos</h3>
                <ol className="text-xs text-zinc-400 list-decimal ml-4 mt-2">
                  <li>Maintain sterility during inoculation.</li>
                  <li>Keep humidity stable during colonization.</li>
                  <li>Avoid sudden light spikes.</li>
                </ol>
              </div>
            </aside>
          </div>

          <div className="mt-4">
            <h3 className="text-sm font-medium">Curva de crecimiento</h3>
            <div className="bg-zinc-900 p-3 rounded-md mt-2">
              <GrowthChart data={growth} />
            </div>
          </div>

        </section>
      </div>

      <footer className="mt-6 text-sm text-zinc-500">Prototipo técnico — ajusta parámetros y usa datos reales para calibración.</footer>
    </div>
  );
}
