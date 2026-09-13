<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>우리만의 동선표 - 오사카 x 교토</title>
<link rel="icon" href="data:,">
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<style>
  html, body { margin: 0; padding: 0; background: #ffffff; }
  #root { min-height: 100vh; }
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel" data-presets="react">

const { useState, useEffect, useRef, useCallback } = React;

// ---------- constants ----------
const CATEGORIES = [
  { id: "cafe", label: "카페", icon: "☕" },
  { id: "food", label: "음식점", icon: "🍜" },
  { id: "activity", label: "액티비티", icon: "🎡" },
  { id: "sight", label: "명소/사찰", icon: "⛩" },
  { id: "shopping", label: "쇼핑", icon: "🛍" },
  { id: "hotel", label: "숙소", icon: "🏨" },
  { id: "subway", label: "지하철역", icon: "🚇" },
  { id: "airport", label: "공항", icon: "✈️" },
  { id: "etc", label: "기타", icon: "📍" },
];
const catIcon = (id) => CATEGORIES.find((c) => c.id === id)?.icon ?? "📍";

const PRIORITIES = [
  { id: "must", label: "필수 코스", color: "#eb5757", bg: "#fbe4e4" },
  { id: "recommend", label: "추천 코스", color: "#448361", bg: "#dbeddb" },
  { id: "optional", label: "여유 코스", color: "#337ea9", bg: "#e7f3f8" },
];
const priority = (id) => PRIORITIES.find((p) => p.id === id) ?? PRIORITIES[1];

const mapsUrl = (name) => `https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(name)}`;

// ---------- helpers ----------
const uid = () =>
  (typeof crypto !== "undefined" && crypto.randomUUID)
    ? crypto.randomUUID()
    : `p_${Date.now()}_${Math.random().toString(16).slice(2)}`;

const dist = (a, b) => Math.hypot(a.x - b.x, a.y - b.y);

function pathLength(seq, byId) {
  let total = 0;
  for (let i = 0; i < seq.length - 1; i++) {
    total += dist(byId[seq[i]], byId[seq[i + 1]]);
  }
  return total;
}

function nearestNeighborOrder(pins) {
  if (pins.length < 2) return pins.map((p) => p.id);
  const byId = Object.fromEntries(pins.map((p) => [p.id, p]));
  const startId = pins.find((p) => p.isStart)?.id ?? pins[0].id;
  const remaining = new Set(pins.map((p) => p.id));
  remaining.delete(startId);
  const seq = [startId];
  let current = startId;
  while (remaining.size) {
    let best = null;
    let bestD = Infinity;
    for (const id of remaining) {
      const d = dist(byId[current], byId[id]);
      if (d < bestD) {
        bestD = d;
        best = id;
      }
    }
    seq.push(best);
    remaining.delete(best);
    current = best;
  }
  return twoOpt(seq, byId);
}

function twoOpt(seq, byId) {
  let improved = true;
  let best = seq.slice();
  while (improved) {
    improved = false;
    for (let i = 1; i < best.length - 1; i++) {
      for (let j = i + 1; j < best.length; j++) {
        const next = best
          .slice(0, i)
          .concat(best.slice(i, j + 1).reverse(), best.slice(j + 1));
        if (pathLength(next, byId) + 0.001 < pathLength(best, byId)) {
          best = next;
          improved = true;
        }
      }
    }
  }
  return best;
}

// ---------- seed data ----------
// x/y are schematic canvas positions (0-600 x 0-400), not real GPS.
const seedDays = () => [
  {
    id: uid(),
    date: "9/22 화",
    label: "난바 · 호리에 · 신사이바시",
    theme: "쇼핑 + 맛집 + 밤거리",
    notice:
      "도착시간이 서로 달라요 — 소연: 에어서울 RS713 / 15:15 도착 (T1). 세혁: 제주항공 / 16:00 도착 (T2). 라피트 탑승역(간사이공항역)은 T1/Aeroplaza 쪽. 동선: 16:00 세혁 T2 도착 → 입국심사/짐 찾기 → T2 밖 무료 셔틀(7~9분) → Aeroplaza 도착 → 2F 간사이공항역에서 T1로 온 소연과 합류. 라피트 열차는 출발 5분 전까지 최대 2회 변경 가능.",
    pins: [
      { id: uid(), name: "간사이 국제공항", note: "소연 15:15(T1) / 세혁 16:00(T2) - 특이사항 참고", x: 560, y: 385, isStart: false, category: "airport", priority: "must" },
      { id: uid(), name: "온야도 노노 난바", note: "체크인, 짐 보관", x: 300, y: 350, isStart: true, category: "hotel", priority: "must" },
      { id: uid(), name: "아메리카무라", note: "", x: 245, y: 300, isStart: false, category: "shopping", priority: "recommend" },
      { id: uid(), name: "Orange Street", note: "⭐ 추천", x: 205, y: 255, isStart: false, category: "shopping", priority: "must" },
      { id: uid(), name: "다이마루 신사이바시점 남관", note: "", x: 275, y: 205, isStart: false, category: "shopping", priority: "recommend" },
      { id: uid(), name: "츠케멘 스즈메", note: "라멘 맛집", x: 250, y: 290, isStart: false, category: "food", priority: "recommend" },
      { id: uid(), name: "도톤보리", note: "야경", x: 320, y: 300, isStart: false, category: "activity", priority: "must" },
      { id: uid(), name: "호젠지 요코초", note: "야식 골목", x: 335, y: 322, isStart: false, category: "food", priority: "optional" },
      { id: uid(), name: "덴덴타운", note: "여유되면 (오타쿠거리)", x: 355, y: 355, isStart: false, category: "shopping", priority: "optional" },
      { id: uid(), name: "신세카이", note: "여유되면 (츠텐카쿠)", x: 345, y: 385, isStart: false, category: "activity", priority: "optional" },
    ],
  },
  {
    id: uid(),
    date: "9/23 수",
    label: "나카노시마 · 우메다 · 오사카성",
    theme: "전시 + 쇼핑 + 야경",
    notice: "",
    pins: [
      { id: uid(), name: "온야도 노노 난바", note: "", x: 300, y: 350, isStart: true, category: "hotel", priority: "must" },
      { id: uid(), name: "오사카 나카노시마 미술관", note: "전시", x: 150, y: 90, isStart: false, category: "sight", priority: "recommend" },
      { id: uid(), name: "Bloom Bouquet", note: "양초 판매점, 나카자키초", x: 175, y: 55, isStart: false, category: "shopping", priority: "optional" },
      { id: uid(), name: "green pepe", note: "빈티지 의류, 나카자키초", x: 190, y: 50, isStart: false, category: "shopping", priority: "optional" },
      { id: uid(), name: "mt lab. OSAKA", note: "마스킹테이프", x: 200, y: 100, isStart: false, category: "shopping", priority: "recommend" },
      { id: uid(), name: "Standard Products", note: "", x: 235, y: 130, isStart: false, category: "shopping", priority: "recommend" },
      { id: uid(), name: "라멘 카모토네기 우메다점", note: "점심", x: 245, y: 145, isStart: false, category: "food", priority: "recommend" },
      { id: uid(), name: "이즈모 루쿠아", note: "민물장어, 점심 대안", x: 250, y: 150, isStart: false, category: "food", priority: "optional" },
      { id: uid(), name: "만다라케 우메다점", note: "여유되면", x: 220, y: 160, isStart: false, category: "shopping", priority: "optional" },
      { id: uid(), name: "키타스시 혼텐", note: "초밥", x: 255, y: 165, isStart: false, category: "food", priority: "recommend" },
      { id: uid(), name: "하나다코", note: "다코야키", x: 240, y: 155, isStart: false, category: "food", priority: "optional" },
      { id: uid(), name: "오사카성", note: "노을 · 야경", x: 410, y: 155, isStart: false, category: "sight", priority: "must" },
      { id: uid(), name: "오사카 아쿠아리움 카이유칸", note: "여유되면 (거리 있음)", x: 480, y: 320, isStart: false, category: "activity", priority: "optional" },
    ],
  },
  {
    id: uid(),
    date: "9/24 목",
    label: "교토 · 후시미 · 히가시야마",
    theme: "교토 핵심 데이트",
    notice: "",
    pins: [
      { id: uid(), name: "교토 우메코지 카덴쇼", note: "체크인, 캐리어 보관", x: 450, y: 350, isStart: true, category: "hotel", priority: "must" },
      { id: uid(), name: "후시미이나리 신사", note: "센본도리이까지", x: 480, y: 250, isStart: false, category: "sight", priority: "must" },
      { id: uid(), name: "규카츠 교토가츠규 후시미이나리신사점", note: "점심", x: 470, y: 230, isStart: false, category: "food", priority: "recommend" },
      { id: uid(), name: "기요미즈데라", note: "", x: 385, y: 205, isStart: false, category: "sight", priority: "must" },
      { id: uid(), name: "시라카와 (기온·행자교 다리)", note: "산넨자카·니넨자카 지나 도보", x: 360, y: 180, isStart: false, category: "sight", priority: "recommend" },
      { id: uid(), name: "야사카 신사", note: "기온", x: 330, y: 195, isStart: false, category: "sight", priority: "must" },
      { id: uid(), name: "히키니쿠토 코메 교토", note: "저녁 대안", x: 335, y: 185, isStart: false, category: "food", priority: "optional" },
      { id: uid(), name: "교토시 교세라 미술관", note: "여유되면 (오카자키)", x: 340, y: 160, isStart: false, category: "sight", priority: "optional" },
    ],
  },
  {
    id: uid(),
    date: "9/25 금",
    label: "아라시야마 · 교토 도심",
    theme: "아라시야마 + 소품 쇼핑",
    notice: "",
    pins: [
      { id: uid(), name: "교토 우메코지 카덴쇼", note: "캐리어 픽업 전 출발", x: 450, y: 350, isStart: true, category: "hotel", priority: "must" },
      { id: uid(), name: "노노미야 신사", note: "대나무숲 근처", x: 85, y: 150, isStart: false, category: "sight", priority: "recommend" },
      { id: uid(), name: "아라시야마 대나무숲", note: "무조건 오전", x: 75, y: 165, isStart: false, category: "sight", priority: "must" },
      { id: uid(), name: "텐류지", note: "", x: 95, y: 185, isStart: false, category: "sight", priority: "recommend" },
      { id: uid(), name: "도게츠교", note: "", x: 100, y: 200, isStart: false, category: "sight", priority: "recommend" },
      { id: uid(), name: "데라마치 상점가", note: "쇼핑", x: 330, y: 260, isStart: false, category: "shopping", priority: "recommend" },
      { id: uid(), name: "신쿄고쿠 상점가", note: "쇼핑", x: 335, y: 270, isStart: false, category: "shopping", priority: "recommend" },
      { id: uid(), name: "규카츠 교토가츠규 교토역전점", note: "점심/저녁", x: 420, y: 335, isStart: false, category: "food", priority: "recommend" },
      { id: uid(), name: "교토 포르타", note: "마지막 기념품", x: 435, y: 330, isStart: false, category: "shopping", priority: "optional" },
    ],
  },
  {
    id: uid(),
    date: "",
    label: "기타 저장 장소 (동선 밖)",
    theme: "구글맵 리스트에는 있지만 이번 동선과 거리가 멀어 별도 확인이 필요한 곳",
    notice: "가나자와 21세기미술관(가나자와시, 편도 2~3시간), 호잔지(나라 이코마, 편도 약 1시간) — 이번 여행에서 뺄지 다음 일본 여행 때 갈지 알려주면 정리할게.",
    pins: [
      { id: uid(), name: "가나자와 21세기미술관", note: "가나자와시 - 거리 멂", x: 300, y: 200, isStart: false, category: "sight", priority: "optional" },
      { id: uid(), name: "호잔지 (이코마)", note: "나라현 이코마 - 거리 있음", x: 300, y: 240, isStart: false, category: "sight", priority: "optional" },
    ],
  },
];

const seqOf = (pins) => pins.map((p) => p.id);
const withDefaults = (p) => ({ category: "etc", priority: "recommend", note: "", ...p });
const withDayDefaults = (d) => ({ notice: "", ...d, pins: (d.pins || []).map(withDefaults) });

function TripRoutePlanner() {
  const [days, setDays] = useState(null);
  const [activeDay, setActiveDay] = useState(0);
  const [placing, setPlacing] = useState(null);
  const [loaded, setLoaded] = useState(false);
  const [saveError, setSaveError] = useState(false);
  const svgRef = useRef(null);

  useEffect(() => {
    try {
      const raw = localStorage.getItem("trip-days");
      if (raw) {
        setDays(JSON.parse(raw).map(withDayDefaults));
      } else {
        setDays(seedDays());
      }
    } catch {
      setDays(seedDays());
    } finally {
      setLoaded(true);
    }
  }, []);

  useEffect(() => {
    if (!loaded || !days) return;
    try {
      localStorage.setItem("trip-days", JSON.stringify(days));
      setSaveError(false);
    } catch {
      setSaveError(true);
    }
  }, [days, loaded]);

  const day = days?.[activeDay];

  const updateDay = useCallback(
    (updater) => {
      setDays((prev) => {
        const next = prev.slice();
        next[activeDay] = updater(next[activeDay]);
        return next;
      });
    },
    [activeDay]
  );

  const handleMapClick = (e) => {
    if (!placing || !placing.name || !svgRef.current) return;
    const rect = svgRef.current.getBoundingClientRect();
    const x = ((e.clientX - rect.left) / rect.width) * 600;
    const y = ((e.clientY - rect.top) / rect.height) * 400;
    updateDay((d) => ({
      ...d,
      pins: [
        ...d.pins,
        {
          id: uid(),
          name: placing.name,
          note: placing.note,
          category: placing.category,
          priority: placing.priority,
          x, y,
          isStart: d.pins.length === 0,
        },
      ],
    }));
    setPlacing(null);
  };

  const removePin = (pinId) => updateDay((d) => ({ ...d, pins: d.pins.filter((p) => p.id !== pinId) }));
  const renamePin = (pinId, name) => updateDay((d) => ({ ...d, pins: d.pins.map((p) => (p.id === pinId ? { ...p, name } : p)) }));
  const noteChange = (pinId, note) => updateDay((d) => ({ ...d, pins: d.pins.map((p) => (p.id === pinId ? { ...p, note } : p)) }));
  const catChange = (pinId, category) => updateDay((d) => ({ ...d, pins: d.pins.map((p) => (p.id === pinId ? { ...p, category } : p)) }));
  const priChange = (pinId, priority) => updateDay((d) => ({ ...d, pins: d.pins.map((p) => (p.id === pinId ? { ...p, priority } : p)) }));
  const setStart = (pinId) => updateDay((d) => ({ ...d, pins: d.pins.map((p) => ({ ...p, isStart: p.id === pinId })) }));

  const move = (pinId, dir) => {
    updateDay((d) => {
      const idx = d.pins.findIndex((p) => p.id === pinId);
      const swapWith = idx + dir;
      if (swapWith < 0 || swapWith >= d.pins.length) return d;
      const pins = d.pins.slice();
      [pins[idx], pins[swapWith]] = [pins[swapWith], pins[idx]];
      return { ...d, pins };
    });
  };

  const optimize = () => {
    updateDay((d) => {
      if (d.pins.length < 3) return d;
      const byId = Object.fromEntries(d.pins.map((p) => [p.id, p]));
      const order = nearestNeighborOrder(d.pins);
      return { ...d, pins: order.map((id) => byId[id]) };
    });
  };

  const addDay = () => {
    setDays((prev) => [...prev, { id: uid(), date: "", label: "새 날짜", theme: "", notice: "", pins: [] }]);
    setActiveDay(days.length);
  };
  const removeDay = () => {
    if (days.length <= 1) return;
    setDays((prev) => prev.filter((_, i) => i !== activeDay));
    setActiveDay((i) => Math.max(0, i - 1));
  };
  const renameDay = (field, value) => updateDay((d) => ({ ...d, [field]: value }));

  if (!loaded || !day) {
    return <div style={{ padding: 40, color: "#9b9a97", fontFamily: "-apple-system, sans-serif" }}>불러오는 중…</div>;
  }

  const rawLength = pathLength(seqOf(day.pins), Object.fromEntries(day.pins.map((p) => [p.id, p])));
  const optimalOrder = day.pins.length > 2 ? nearestNeighborOrder(day.pins) : seqOf(day.pins);
  const optimalLength = pathLength(optimalOrder, Object.fromEntries(day.pins.map((p) => [p.id, p])));
  const improvement = rawLength > 0 ? Math.max(0, Math.round((1 - optimalLength / rawLength) * 100)) : 0;
  const alreadyOptimal = seqOf(day.pins).join(",") === optimalOrder.join(",");

  return (
    <div className="rp-root">
      <style>{`
        * { box-sizing: border-box; }
        .rp-root {
          min-height: 100%;
          background: #ffffff;
          color: #37352f;
          font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Apple SD Gothic Neo", "Malgun Gothic", Helvetica, Arial, sans-serif;
          padding: 48px 64px 60px;
          max-width: 900px;
          margin: 0 auto;
        }
        @media (max-width: 700px) { .rp-root { padding: 28px 20px 40px; } }

        .rp-icon { font-size: 44px; line-height: 1; margin-bottom: 4px; }
        .rp-title { font-size: 34px; font-weight: 700; margin: 0 0 6px; letter-spacing: -0.015em; color: #37352f; }
        .rp-subtitle { font-size: 14px; color: #787774; margin: 0; line-height: 1.5; }

        .rp-props { display: flex; flex-wrap: wrap; gap: 6px 18px; margin: 18px 0 4px; padding: 10px 0; border-top: 1px solid #e9e9e7; }
        .rp-prop { display: flex; align-items: center; gap: 6px; font-size: 13px; color: #787774; }
        .rp-prop b { color: #9b9a97; font-weight: 500; min-width: 52px; }
        .rp-pill { display: inline-flex; align-items: center; gap: 5px; padding: 2px 8px; border-radius: 4px; font-size: 12.5px; font-weight: 500; }

        .rp-tabs { display: flex; flex-wrap: wrap; gap: 2px; margin: 22px 0 6px; border-bottom: 1px solid #e9e9e7; }
        .rp-tab {
          border: none; background: transparent; color: #787774; padding: 8px 12px; font-size: 13.5px; font-weight: 500;
          cursor: pointer; border-radius: 6px 6px 0 0; display: flex; align-items: center; gap: 6px;
          border-bottom: 2px solid transparent; margin-bottom: -1px;
        }
        .rp-tab:hover { background: #f7f7f5; }
        .rp-tab.active { color: #37352f; font-weight: 600; border-bottom: 2px solid #37352f; }
        .rp-addday {
          border: none; background: transparent; color: #9b9a97; padding: 8px 12px; font-size: 13px; cursor: pointer; border-radius: 6px;
        }
        .rp-addday:hover { background: #f7f7f5; color: #37352f; }

        .rp-daymeta { display: flex; align-items: baseline; gap: 4px; margin: 18px 0 10px; flex-wrap: wrap; }
        .rp-daymeta input.label {
          font-size: 22px; font-weight: 700; background: transparent; border: none; color: #37352f;
          padding: 3px 4px; border-radius: 4px; min-width: 200px;
        }
        .rp-daymeta input.label:hover, .rp-daymeta input.label:focus { background: #f7f7f5; outline: none; }
        .rp-daymeta input.theme {
          font-size: 13px; font-weight: 400; background: transparent; border: none; color: #9b9a97;
          padding: 3px 6px; border-radius: 4px; flex: 1; min-width: 140px;
        }
        .rp-daymeta input.theme:hover, .rp-daymeta input.theme:focus { background: #f7f7f5; outline: none; }
        .rp-daydel { background: none; border: none; color: #9b9a97; font-size: 12px; cursor: pointer; padding: 4px 8px; border-radius: 4px; }
        .rp-daydel:hover { background: #fbe4e4; color: #eb5757; }

        .rp-notice { margin: 4px 0 20px; background: #fbf3db; border-radius: 4px; padding: 12px 14px; display: flex; gap: 10px; align-items: flex-start; }
        .rp-notice .ic { font-size: 16px; line-height: 1.4; }
        .rp-notice textarea {
          flex: 1; width: 100%; background: transparent; border: none; color: #4a4640; font-size: 13.5px; font-weight: 400;
          line-height: 1.6; resize: vertical; min-height: 42px; font-family: inherit;
        }
        .rp-notice textarea:focus { outline: none; }

        .rp-grid { display: grid; grid-template-columns: 1.15fr 0.85fr; gap: 20px; align-items: start; }
        @media (max-width: 760px) { .rp-grid { grid-template-columns: 1fr; } }

        .rp-map-card { background: #ffffff; border: 1px solid #e9e9e7; border-radius: 8px; padding: 8px; overflow: hidden; }
        .rp-map-hint { font-size: 12px; font-weight: 400; color: #9b9a97; padding: 6px 8px 10px; }
        .rp-placingbar { background: #37352f; color: #fff; font-size: 12.5px; font-weight: 500; padding: 8px 12px; margin-bottom: 8px; border-radius: 6px; display: flex; justify-content: space-between; align-items: center; }
        .rp-placingbar button { background: rgba(255,255,255,0.15); border: none; color: #fff; font-size: 11.5px; padding: 4px 10px; border-radius: 4px; cursor: pointer; }
        svg.rp-svg { width: 100%; height: auto; display: block; cursor: default; border-radius: 4px; }
        svg.rp-svg.placing { cursor: crosshair; }

        .rp-panel { background: #ffffff; border: 1px solid #e9e9e7; border-radius: 8px; padding: 18px; }
        .rp-addform { display: flex; flex-direction: column; gap: 6px; margin-bottom: 18px; }
        .rp-addform input {
          background: #f7f7f5; border: 1px solid transparent; color: #37352f; padding: 8px 10px; border-radius: 6px; font-size: 13.5px; width: 100%;
        }
        .rp-addform input::placeholder { color: #9b9a97; }
        .rp-addform input:focus { outline: none; border-color: #37352f22; background: #fff; box-shadow: 0 0 0 1px #37352f22; }
        .rp-selectrow { display: flex; gap: 6px; }
        .rp-selectrow select { flex: 1; background: #f7f7f5; border: 1px solid transparent; color: #37352f; padding: 7px 6px; border-radius: 6px; font-size: 12.5px; }
        .rp-selectrow select:focus { outline: none; }
        .rp-addbtn { background: #37352f; color: #fff; border: none; padding: 8px 14px; border-radius: 6px; font-size: 13px; font-weight: 500; cursor: pointer; align-self: flex-start; }
        .rp-addbtn:hover:not(:disabled) { background: #52504a; }
        .rp-addbtn:disabled { opacity: 0.35; cursor: default; }

        .rp-list { display: flex; flex-direction: column; margin-bottom: 16px; }
        .rp-row { display: flex; align-items: center; gap: 8px; padding: 8px 4px; border-bottom: 1px solid #f0f0ee; }
        .rp-row:hover { background: #fbfbfa; }
        .rp-badge { width: 26px; height: 26px; flex: none; background: #f7f7f5; border-radius: 6px; font-size: 14px; display: flex; align-items: center; justify-content: center; position: relative; }
        .rp-badge .num { position: absolute; bottom: -5px; right: -5px; background: #37352f; color: #fff; font-size: 8.5px; font-weight: 700; width: 14px; height: 14px; border-radius: 50%; display: flex; align-items: center; justify-content: center; }
        .rp-row.start .rp-badge { background: #eeece9; }
        .rp-rowfields { flex: 1; min-width: 0; display: flex; flex-direction: column; gap: 2px; }
        .rp-rowfields input { background: transparent; border: none; color: #37352f; font-size: 13.5px; font-weight: 500; padding: 1px 2px; width: 100%; border-radius: 4px; }
        .rp-rowfields input.note { font-size: 12px; color: #9b9a97; font-weight: 400; }
        .rp-rowfields input:hover, .rp-rowfields input:focus { background: #f7f7f5; outline: none; }
        .rp-rowselects { display: flex; gap: 4px; margin-top: 2px; }
        .rp-rowselects select { font-size: 10.5px; font-weight: 500; border: none; background: #f7f7f5; color: #787774; padding: 2px 4px; border-radius: 4px; }
        .rp-rowbtns { display: flex; gap: 1px; flex: none; }
        .rp-iconbtn { background: none; border: none; color: #b9b8b4; font-size: 12.5px; cursor: pointer; padding: 4px 6px; border-radius: 4px; }
        .rp-iconbtn:hover { color: #37352f; background: #f0f0ee; }
        .rp-iconbtn.star.active { color: #dfab01; }
        .rp-iconbtn.maplink { color: #337ea9; }
        .rp-iconbtn.maplink:hover { background: #e7f3f8; }

        .rp-optimize {
          width: 100%; background: #f7f7f5; border: 1px solid transparent; color: #37352f;
          padding: 10px; border-radius: 6px; font-size: 13px; font-weight: 500; cursor: pointer; margin-bottom: 14px;
        }
        .rp-optimize:hover:not(:disabled) { background: #eeece9; }
        .rp-optimize:disabled { opacity: 0.35; cursor: default; }

        .rp-stats { font-size: 12px; color: #9b9a97; font-weight: 400; line-height: 1.8; border-top: 1px solid #e9e9e7; padding-top: 10px; }
        .rp-stats b { color: #37352f; font-weight: 600; }
        .rp-savewarn { font-size: 11.5px; color: #eb5757; margin-top: 6px; font-weight: 500; }
      `}</style>

      <div className="rp-icon">🗾</div>
      <h1 className="rp-title">우리만의 동선표</h1>
      <p className="rp-subtitle">주소를 추가하면 날짜별로 가장 덜 걷는 순서를 계산해줘요.</p>

      <div className="rp-props">
        <div className="rp-prop"><b>우선순위</b>
          {PRIORITIES.map((p) => (
            <span className="rp-pill" key={p.id} style={{ background: p.bg, color: p.color }}>{p.label}</span>
          ))}
        </div>
        <div className="rp-prop"><b>장소</b>
          {CATEGORIES.filter((c) => c.id !== "etc").map((c) => (
            <span key={c.id} style={{ marginRight: 2 }}>{c.icon} {c.label}</span>
          ))}
        </div>
      </div>

      <div className="rp-tabs">
        {days.map((d, i) => (
          <button key={d.id} className={"rp-tab" + (i === activeDay ? " active" : "")} onClick={() => { setActiveDay(i); setPlacing(null); }}>
            {d.date || `Day ${i + 1}`}
          </button>
        ))}
        <button className="rp-addday" onClick={addDay}>+ 날짜 추가</button>
      </div>

      <div className="rp-daymeta">
        <input className="label" value={day.label} onChange={(e) => renameDay("label", e.target.value)} />
        <input className="theme" value={day.theme} placeholder="이 날의 테마" onChange={(e) => renameDay("theme", e.target.value)} />
        {days.length > 1 && <button className="rp-daydel" onClick={removeDay}>삭제</button>}
      </div>

      <div className="rp-notice">
        <span className="ic">⚠️</span>
        <textarea
          value={day.notice}
          placeholder="이 날짜의 특이사항을 적어두세요 (예: 도착시간이 다른 경우, 예약 필요 등)"
          onChange={(e) => renameDay("notice", e.target.value)}
        />
      </div>

      <div className="rp-grid">
        <div className="rp-map-card">
          {placing && (
            <div className="rp-placingbar">
              <span>"{placing.name || "이름 없음"}" 위치를 지도에서 탭하세요</span>
              <button onClick={() => setPlacing(null)}>취소</button>
            </div>
          )}
          {!placing && <div className="rp-map-hint">대략적인 위치 배치용 지도 · 선 색 = 목적지 우선순위 · 목록의 🗺 버튼으로 실제 구글맵 열기</div>}
          <svg ref={svgRef} className={"rp-svg" + (placing ? " placing" : "")} viewBox="0 0 600 400" onClick={handleMapClick}>
            <defs>
              <pattern id="grid" width="24" height="24" patternUnits="userSpaceOnUse">
                <circle cx="1" cy="1" r="1" fill="rgba(55,53,47,0.10)" />
              </pattern>
              {PRIORITIES.map((p) => (
                <marker key={p.id} id={"arrow-" + p.id} viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
                  <path d="M0,0 L10,5 L0,10 z" fill={p.color} />
                </marker>
              ))}
            </defs>
            <rect x="0" y="0" width="600" height="400" fill="url(#grid)" />

            {day.pins.length > 1 &&
              day.pins.slice(0, -1).map((p, i) => {
                const n = day.pins[i + 1];
                const c = priority(n.priority).color;
                return (
                  <line key={p.id + "-" + n.id} x1={p.x} y1={p.y} x2={n.x} y2={n.y}
                    stroke={c} strokeWidth="2" strokeDasharray="5 4" markerEnd={`url(#arrow-${n.priority})`} opacity="0.85" />
                );
              })}

            {day.pins.map((p, i) => (
              <g key={p.id} transform={`translate(${p.x},${p.y})`}>
                <circle r={p.isStart ? 13 : 11} fill="#fff" stroke={p.isStart ? "#37352f" : priority(p.priority).color} strokeWidth={p.isStart ? 3 : 2.5} />
                <text textAnchor="middle" dy="5" fontSize="12">{p.isStart ? "🏨" : catIcon(p.category)}</text>
                <circle cx="9" cy="-9" r="6.5" fill="#37352f" />
                <text x="9" y="-9" textAnchor="middle" dy="2.5" fontSize="7.5" fontWeight="700" fill="#fff">{i + 1}</text>
                <text x="0" y={p.isStart ? -21 : -18} textAnchor="middle" fontSize="11" fontWeight="600" fill="#37352f">
                  {p.name.length > 12 ? p.name.slice(0, 12) + "…" : p.name}
                </text>
              </g>
            ))}
          </svg>
        </div>

        <div className="rp-panel">
          <div className="rp-addform">
            <input
              placeholder="장소 이름 / 주소"
              value={placing?.name ?? ""}
              onChange={(e) => setPlacing((p) => ({ name: e.target.value, note: p?.note ?? "", category: p?.category ?? "etc", priority: p?.priority ?? "recommend" }))}
              onFocus={() => setPlacing((p) => p ?? { name: "", note: "", category: "etc", priority: "recommend" })}
            />
            <input
              placeholder="메모 (선택, 예약 여부 등)"
              value={placing?.note ?? ""}
              onChange={(e) => setPlacing((p) => ({ ...(p ?? { name: "", category: "etc", priority: "recommend" }), note: e.target.value }))}
            />
            <div className="rp-selectrow">
              <select
                value={placing?.category ?? "etc"}
                onChange={(e) => setPlacing((p) => ({ ...(p ?? { name: "", note: "", priority: "recommend" }), category: e.target.value }))}
              >
                {CATEGORIES.map((c) => <option key={c.id} value={c.id}>{c.icon} {c.label}</option>)}
              </select>
              <select
                value={placing?.priority ?? "recommend"}
                onChange={(e) => setPlacing((p) => ({ ...(p ?? { name: "", note: "", category: "etc" }), priority: e.target.value }))}
              >
                {PRIORITIES.map((p) => <option key={p.id} value={p.id}>{p.label}</option>)}
              </select>
            </div>
            <button className="rp-addbtn" disabled={!placing?.name}>
              {placing?.name ? "지도를 탭해 위치 지정 →" : "이름을 입력하세요"}
            </button>
          </div>

          <button className="rp-optimize" disabled={day.pins.length < 3 || alreadyOptimal} onClick={optimize}>
            {day.pins.length < 3 ? "장소가 3곳 이상일 때 최적화 가능" : alreadyOptimal ? "이미 최적 순서예요" : `최적 동선으로 재배열 (약 ${improvement}% 단축)`}
          </button>

          <div className="rp-list">
            {day.pins.map((p, i) => (
              <div className={"rp-row" + (p.isStart ? " start" : "")} key={p.id}>
                <div className="rp-badge">
                  {p.isStart ? "🏨" : catIcon(p.category)}
                  <span className="num">{i + 1}</span>
                </div>
                <div className="rp-rowfields">
                  <input className="name" value={p.name} onChange={(e) => renamePin(p.id, e.target.value)} />
                  <input className="note" value={p.note} placeholder="메모" onChange={(e) => noteChange(p.id, e.target.value)} />
                  <div className="rp-rowselects">
                    <select value={p.category} onChange={(e) => catChange(p.id, e.target.value)}>
                      {CATEGORIES.map((c) => <option key={c.id} value={c.id}>{c.icon} {c.label}</option>)}
                    </select>
                    <select value={p.priority} onChange={(e) => priChange(p.id, e.target.value)}>
                      {PRIORITIES.map((pr) => <option key={pr.id} value={pr.id}>{pr.label}</option>)}
                    </select>
                  </div>
                </div>
                <div className="rp-rowbtns">
                  <button className="rp-iconbtn maplink" onClick={() => window.open(mapsUrl(p.name), "_blank")} title="구글맵에서 열기">🗺</button>
                  <button className="rp-iconbtn" onClick={() => move(p.id, -1)} title="위로">↑</button>
                  <button className="rp-iconbtn" onClick={() => move(p.id, 1)} title="아래로">↓</button>
                  <button className={"rp-iconbtn star" + (p.isStart ? " active" : "")} onClick={() => setStart(p.id)} title="출발지(숙소)로 지정">★</button>
                  <button className="rp-iconbtn" onClick={() => removePin(p.id)} title="삭제">✕</button>
                </div>
              </div>
            ))}
            {day.pins.length === 0 && (
              <p style={{ fontSize: 12.5, color: "#9b9a97" }}>아직 추가된 장소가 없어요. 위에서 이름을 입력하고 지도를 탭해보세요.</p>
            )}
          </div>

          <div className="rp-stats">
            현재 순서 이동 거리: <b>{rawLength.toFixed(0)}</b> (상대 단위)<br />
            최적 순서 이동 거리: <b>{optimalLength.toFixed(0)}</b>
            {saveError && <div className="rp-savewarn">저장에 실패했어요. 이 세션에서는 계속 사용할 수 있어요.</div>}
          </div>
        </div>
      </div>
    </div>
  );
}


const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<TripRoutePlanner />);
</script>
</body>
</html>
