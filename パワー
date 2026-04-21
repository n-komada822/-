import React, { useEffect, useMemo, useRef, useState } from "react";
import html2canvas from "html2canvas";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import {
  Plus,
  Trash2,
  Download,
  CalendarDays,
  ChevronLeft,
  ChevronRight,
  Save,
  Dumbbell,
  ClipboardList,
  History,
  Smartphone,
  Share2,
} from "lucide-react";

const STORAGE_KEY = "workout-rpe-tool-mobile-v1";
const EXERCISE_STORAGE_KEY = "workout-rpe-tool-mobile-exercises-v1";
const DEFAULT_EXERCISES = [
  "スクワット",
  "ベンチプレス",
  "デッドリフト",
  "ナローハイバーSQ",
  "Pベンチ",
  "ミリタリープレス",
  "サイドレイズ",
];

function uid() {
  return Math.random().toString(36).slice(2) + Date.now().toString(36);
}

function emptySet() {
  return {
    id: uid(),
    weight: "",
    reps: "",
    plannedRpe: "",
    actualRpe: "",
  };
}

function emptyExercise(name = "") {
  return {
    id: uid(),
    name,
    sets: [emptySet()],
  };
}

function todayInputValue() {
  const d = new Date();
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth() + 1).padStart(2, "0");
  const dd = String(d.getDate()).padStart(2, "0");
  return `${yyyy}-${mm}-${dd}`;
}

function displayDate(isoDate) {
  if (!isoDate) return "";
  return isoDate.replaceAll("-", "/");
}

function formatDateJP(dateLike) {
  const d = new Date(dateLike);
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth() + 1).padStart(2, "0");
  const dd = String(d.getDate()).padStart(2, "0");
  return `${yyyy}/${mm}/${dd}`;
}

function numeric(value) {
  const n = Number(value);
  return Number.isFinite(n) ? n : 0;
}

function calcSetVolume(set) {
  return numeric(set.weight) * numeric(set.reps);
}

function calcExerciseVolume(exercise) {
  return exercise.sets.reduce((sum, set) => sum + calcSetVolume(set), 0);
}

function calcSessionVolume(exercises) {
  return exercises.reduce((sum, exercise) => sum + calcExerciseVolume(exercise), 0);
}

function calcEstimated1RM(weight, reps, rpe) {
  const w = numeric(weight);
  const r = numeric(reps);
  const actual = numeric(rpe);
  if (!w || !r || !actual) return "-";
  const rir = Math.max(0, 10 - actual);
  const e1rm = w * (1 + (r + rir) / 30);
  return `${e1rm.toFixed(1)} kg`;
}

function overPlanned(planned, actual) {
  return numeric(actual) > numeric(planned) && numeric(planned) > 0;
}

function sameDay(a, b) {
  const da = new Date(a);
  const db = new Date(b);
  return (
    da.getFullYear() === db.getFullYear() &&
    da.getMonth() === db.getMonth() &&
    da.getDate() === db.getDate()
  );
}

function startOfWeek(date) {
  const d = new Date(date);
  const day = d.getDay();
  const diff = day === 0 ? -6 : 1 - day;
  d.setDate(d.getDate() + diff);
  d.setHours(0, 0, 0, 0);
  return d;
}

function weekDays(baseDate) {
  const start = startOfWeek(baseDate);
  return Array.from({ length: 7 }, (_, i) => {
    const d = new Date(start);
    d.setDate(start.getDate() + i);
    return d;
  });
}

function monthGrid(baseDate) {
  const year = baseDate.getFullYear();
  const month = baseDate.getMonth();
  const first = new Date(year, month, 1);
  const last = new Date(year, month + 1, 0);
  const start = startOfWeek(first);
  const end = new Date(last);
  const weekday = end.getDay() === 0 ? 7 : end.getDay();
  end.setDate(end.getDate() + (7 - weekday));
  end.setHours(0, 0, 0, 0);

  const days = [];
  const cursor = new Date(start);
  while (cursor <= end) {
    days.push(new Date(cursor));
    cursor.setDate(cursor.getDate() + 1);
  }
  return days;
}

function SessionSummaryCard({ session }) {
  return (
    <div
      className="w-full bg-black text-white"
      style={{ fontFamily: "system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif" }}
    >
      <div className="mx-auto w-full max-w-[1120px] px-3 py-4 sm:px-6 sm:py-6">
        <div className="mb-4 border-b border-white/20 pb-3 sm:mb-5 sm:pb-4">
          <div className="text-[24px] font-bold tracking-tight sm:text-[38px] md:text-[56px]">
            {displayDate(session.date)} WorkOut
          </div>
        </div>

        <div className="space-y-3 sm:space-y-4">
          {session.exercises.map((exercise) => (
            <div key={exercise.id} className="rounded-[18px] border border-white/35 px-3 py-3 sm:rounded-[22px] sm:px-6 sm:py-4">
              <div className="mb-2 text-[18px] font-bold sm:mb-3 sm:text-[28px] md:text-[34px]">
                {exercise.name || "種目名未入力"}
              </div>

              <div className="grid grid-cols-[0.7fr_1.2fr_0.9fr_0.9fr_0.9fr] gap-2 border-b border-white/20 pb-2 text-[10px] text-white/70 sm:grid-cols-[0.8fr_1.4fr_1.2fr_1fr_1fr] sm:gap-3 sm:text-base md:text-[18px]">
                <div>Set</div>
                <div>重量</div>
                <div>回数</div>
                <div>予定</div>
                <div>実際</div>
              </div>

              <div className="space-y-1.5 pt-2 text-[12px] sm:space-y-2 sm:text-[20px] md:text-[24px]">
                {exercise.sets.map((set, index) => {
                  const isHigh = overPlanned(set.plannedRpe, set.actualRpe);
                  return (
                    <div
                      key={set.id}
                      className="grid grid-cols-[0.7fr_1.2fr_0.9fr_0.9fr_0.9fr] items-center gap-2 sm:grid-cols-[0.8fr_1.4fr_1.2fr_1fr_1fr] sm:gap-3"
                    >
                      <div>{index + 1}</div>
                      <div>{set.weight || "-"}kg</div>
                      <div>{set.reps || "-"}rep</div>
                      <div>{set.plannedRpe || "-"}</div>
                      <div className={isHigh ? "font-bold text-red-500" : "text-white"}>{set.actualRpe || "-"}</div>
                    </div>
                  );
                })}
              </div>
            </div>
          ))}
        </div>

        {(session.memo || session.totalVolume) && (
          <div className="mt-3 rounded-[18px] border border-white/35 px-3 py-3 sm:mt-4 sm:px-6 sm:py-4">
            <div className="mb-2 flex flex-wrap items-center gap-3 text-xs sm:gap-4 sm:text-lg md:text-[20px]">
              <div className="font-bold text-white">メモ</div>
              <div className="text-white/70">総ボリューム : {session.totalVolume} kg</div>
            </div>
            <div className="whitespace-pre-wrap text-[12px] text-white/90 sm:text-[18px] md:text-[22px]">
              {session.memo || "-"}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}

function ExerciseInputCard({
  exercise,
  exerciseMaster,
  updateExerciseName,
  removeExerciseBlock,
  addSetToExercise,
  removeSetFromExercise,
  updateSet,
}) {
  return (
    <div className="rounded-3xl border border-zinc-800 bg-zinc-950 p-3 space-y-4 sm:p-4">
      <div className="space-y-3">
        <select
          value={exercise.name}
          onChange={(e) => updateExerciseName(exercise.id, e.target.value)}
          className="h-11 w-full rounded-xl border border-zinc-700 bg-zinc-900 px-3 text-sm text-zinc-100 outline-none"
        >
          {exerciseMaster.map((name) => (
            <option key={name} value={name}>
              {name}
            </option>
          ))}
        </select>

        <div className="flex items-center justify-between gap-3">
          <div className="text-sm text-zinc-400">種目Vol: {calcExerciseVolume(exercise)} kg</div>
          <Button onClick={() => removeExerciseBlock(exercise.id)} variant="ghost" className="rounded-2xl px-3 text-zinc-300">
            <Trash2 className="mr-2 h-4 w-4" />種目削除
          </Button>
        </div>
      </div>

      <div className="space-y-3">
        {exercise.sets.map((set, setIndex) => {
          const isHigh = overPlanned(set.plannedRpe, set.actualRpe);
          return (
            <div key={set.id} className="rounded-2xl border border-zinc-800 bg-zinc-900 p-3 space-y-3">
              <div className="flex items-center justify-between">
                <div className="rounded-xl bg-zinc-800 px-3 py-2 text-sm font-medium">Set {setIndex + 1}</div>
                <Button
                  onClick={() => removeSetFromExercise(exercise.id, set.id)}
                  variant="ghost"
                  className="rounded-2xl px-3 text-zinc-300"
                >
                  <Trash2 className="h-4 w-4" />
                </Button>
              </div>

              <div className="grid grid-cols-2 gap-3">
                <div>
                  <Label className="text-xs text-zinc-400">重量</Label>
                  <Input
                    inputMode="decimal"
                    value={set.weight}
                    onChange={(e) => updateSet(exercise.id, set.id, "weight", e.target.value)}
                    placeholder="157.5"
                    className="mt-2 h-11 border-zinc-700 bg-zinc-950 text-zinc-100 text-base"
                  />
                </div>
                <div>
                  <Label className="text-xs text-zinc-400">回数</Label>
                  <Input
                    inputMode="numeric"
                    value={set.reps}
                    onChange={(e) => updateSet(exercise.id, set.id, "reps", e.target.value)}
                    placeholder="6"
                    className="mt-2 h-11 border-zinc-700 bg-zinc-950 text-zinc-100 text-base"
                  />
                </div>
                <div>
                  <Label className="text-xs text-zinc-400">予定RPE</Label>
                  <Input
                    inputMode="decimal"
                    value={set.plannedRpe}
                    onChange={(e) => updateSet(exercise.id, set.id, "plannedRpe", e.target.value)}
                    placeholder="8"
                    className="mt-2 h-11 border-zinc-700 bg-zinc-950 text-zinc-100 text-base"
                  />
                </div>
                <div>
                  <Label className="text-xs text-zinc-400">実際RPE</Label>
                  <Input
                    inputMode="decimal"
                    value={set.actualRpe}
                    onChange={(e) => updateSet(exercise.id, set.id, "actualRpe", e.target.value)}
                    placeholder="8.5"
                    className="mt-2 h-11 border-zinc-700 bg-zinc-950 text-zinc-100 text-base"
                  />
                </div>
              </div>

              <div className="rounded-2xl bg-zinc-800/60 px-4 py-3 text-sm text-zinc-300 space-y-1">
                <div>
                  e1RM: <span className="font-semibold text-zinc-100">{calcEstimated1RM(set.weight, set.reps, set.actualRpe)}</span>
                </div>
                <div>
                  Volume: <span className="font-semibold text-zinc-100">{calcSetVolume(set)} kg</span>
                </div>
                <div>
                  判定: <span className={isHigh ? "font-semibold text-red-400" : "font-semibold text-zinc-100"}>{isHigh ? "予定超過" : "予定内"}</span>
                </div>
              </div>
            </div>
          );
        })}
      </div>

      <Button
        onClick={() => addSetToExercise(exercise.id)}
        variant="outline"
        className="h-11 w-full rounded-2xl border-zinc-700 bg-zinc-950 text-zinc-100"
      >
        <Plus className="mr-2 h-4 w-4" />セット追加
      </Button>
    </div>
  );
}

export default function WorkoutRpeTool() {
  const captureRef = useRef(null);
  const [exerciseMaster, setExerciseMaster] = useState(DEFAULT_EXERCISES);
  const [newExerciseName, setNewExerciseName] = useState("");
  const [sessionDate, setSessionDate] = useState(todayInputValue());
  const [memo, setMemo] = useState("");
  const [exercises, setExercises] = useState([emptyExercise(DEFAULT_EXERCISES[0])]);
  const [history, setHistory] = useState([]);
  const [calendarMode, setCalendarMode] = useState("month");
  const [calendarDate, setCalendarDate] = useState(new Date());
  const [isExporting, setIsExporting] = useState(false);
  const [showInstallGuide, setShowInstallGuide] = useState(false);

  useEffect(() => {
    document.title = "筋トレRPEログ";

    const setMetaTag = (selector, attrs) => {
      let el = document.querySelector(selector);
      if (!el) {
        el = document.createElement(selector.startsWith("link") ? "link" : "meta");
        document.head.appendChild(el);
      }
      Object.entries(attrs).forEach(([key, value]) => el.setAttribute(key, value));
    };

    setMetaTag('meta[name="theme-color"]', { name: 'theme-color', content: '#09090b' });
    setMetaTag('meta[name="apple-mobile-web-app-capable"]', { name: 'apple-mobile-web-app-capable', content: 'yes' });
    setMetaTag('meta[name="apple-mobile-web-app-status-bar-style"]', { name: 'apple-mobile-web-app-status-bar-style', content: 'black-translucent' });
    setMetaTag('meta[name="apple-mobile-web-app-title"]', { name: 'apple-mobile-web-app-title', content: '筋トレRPEログ' });
    setMetaTag('meta[name="mobile-web-app-capable"]', { name: 'mobile-web-app-capable', content: 'yes' });

    const manifest = {
      name: '筋トレRPEログ',
      short_name: '筋トレRPE',
      display: 'standalone',
      background_color: '#000000',
      theme_color: '#09090b',
      start_url: '.',
      icons: [],
    };
    const manifestBlob = new Blob([JSON.stringify(manifest)], { type: 'application/json' });
    const manifestUrl = URL.createObjectURL(manifestBlob);
    setMetaTag('link[rel="manifest"]', { rel: 'manifest', href: manifestUrl });

    return () => {
      URL.revokeObjectURL(manifestUrl);
    };
  }, []);

  useEffect(() => {
    const savedHistory = localStorage.getItem(STORAGE_KEY);
    const savedExercises = localStorage.getItem(EXERCISE_STORAGE_KEY);
    if (savedHistory) {
      try {
        const parsed = JSON.parse(savedHistory);
        if (Array.isArray(parsed)) setHistory(parsed);
      } catch {}
    }
    if (savedExercises) {
      try {
        const parsed = JSON.parse(savedExercises);
        if (Array.isArray(parsed) && parsed.length) setExerciseMaster(parsed);
      } catch {}
    }
  }, []);

  useEffect(() => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(history));
  }, [history]);

  useEffect(() => {
    localStorage.setItem(EXERCISE_STORAGE_KEY, JSON.stringify(exerciseMaster));
  }, [exerciseMaster]);

  const sessionVolume = useMemo(() => calcSessionVolume(exercises), [exercises]);

  const currentSession = useMemo(
    () => ({
      id: uid(),
      date: sessionDate,
      memo,
      exercises,
      totalVolume: sessionVolume,
    }),
    [sessionDate, memo, exercises, sessionVolume]
  );

  const addExerciseMaster = () => {
    const name = newExerciseName.trim();
    if (!name) return;
    if (!exerciseMaster.includes(name)) {
      setExerciseMaster((prev) => [...prev, name]);
    }
    setNewExerciseName("");
  };

  const addExerciseBlock = () => {
    setExercises((prev) => [...prev, emptyExercise(exerciseMaster[0] || "")]);
  };

  const removeExerciseBlock = (exerciseId) => {
    setExercises((prev) => (prev.length === 1 ? prev : prev.filter((e) => e.id !== exerciseId)));
  };

  const updateExerciseName = (exerciseId, name) => {
    setExercises((prev) => prev.map((e) => (e.id === exerciseId ? { ...e, name } : e)));
  };

  const addSetToExercise = (exerciseId) => {
    setExercises((prev) =>
      prev.map((e) => (e.id === exerciseId ? { ...e, sets: [...e.sets, emptySet()] } : e))
    );
  };

  const removeSetFromExercise = (exerciseId, setId) => {
    setExercises((prev) =>
      prev.map((e) => {
        if (e.id !== exerciseId) return e;
        if (e.sets.length === 1) return e;
        return { ...e, sets: e.sets.filter((s) => s.id !== setId) };
      })
    );
  };

  const updateSet = (exerciseId, setId, field, value) => {
    setExercises((prev) =>
      prev.map((e) => {
        if (e.id !== exerciseId) return e;
        return {
          ...e,
          sets: e.sets.map((s) => (s.id === setId ? { ...s, [field]: value } : s)),
        };
      })
    );
  };

  const clearSession = () => {
    setSessionDate(todayInputValue());
    setMemo("");
    setExercises([emptyExercise(exerciseMaster[0] || "")]);
  };

  const saveSession = () => {
    const cleanedExercises = exercises
      .map((exercise) => ({
        ...exercise,
        name: exercise.name.trim(),
        sets: exercise.sets.filter(
          (set) =>
            set.weight !== "" ||
            set.reps !== "" ||
            set.plannedRpe !== "" ||
            set.actualRpe !== ""
        ),
      }))
      .filter((exercise) => exercise.name && exercise.sets.length > 0);

    if (!cleanedExercises.length) return;

    const entry = {
      id: uid(),
      date: sessionDate,
      memo: memo.trim(),
      exercises: cleanedExercises,
      totalVolume: calcSessionVolume(cleanedExercises),
      createdAt: new Date().toISOString(),
    };

    setHistory((prev) => [entry, ...prev]);
  };

  const loadSession = (entry) => {
    setSessionDate(entry.date);
    setMemo(entry.memo || "");
    setExercises(
      entry.exercises.map((exercise) => ({
        ...exercise,
        id: uid(),
        sets: exercise.sets.map((set) => ({ ...set, id: uid() })),
      }))
    );
    window.scrollTo({ top: 0, behavior: "smooth" });
  };

  const deleteSession = (id) => {
    setHistory((prev) => prev.filter((entry) => entry.id !== id));
  };

  const exportImage = async () => {
    if (!captureRef.current) return;
    setIsExporting(true);
    try {
      const canvas = await html2canvas(captureRef.current, {
        backgroundColor: "#000000",
        scale: 2,
        useCORS: true,
      });
      const link = document.createElement("a");
      link.download = `${displayDate(sessionDate)}_workout.png`;
      link.href = canvas.toDataURL("image/png");
      link.click();
    } finally {
      setIsExporting(false);
    }
  };

  const periodDays = useMemo(
    () => (calendarMode === "month" ? monthGrid(calendarDate) : weekDays(calendarDate)),
    [calendarMode, calendarDate]
  );

  const entriesByDate = useMemo(() => {
    const map = new Map();
    for (const day of periodDays) {
      const key = formatDateJP(day);
      map.set(
        key,
        history.filter((entry) => sameDay(entry.date, day))
      );
    }
    return map;
  }, [history, periodDays]);

  const periodSummary = useMemo(() => {
    const entries = history.filter((entry) => periodDays.some((day) => sameDay(entry.date, day)));
    const totalVolume = entries.reduce((sum, entry) => sum + numeric(entry.totalVolume), 0);
    const exerciseCount = entries.reduce((sum, entry) => sum + entry.exercises.length, 0);
    const setCount = entries.reduce(
      (sum, entry) => sum + entry.exercises.reduce((s, e) => s + e.sets.length, 0),
      0
    );
    return { entries: entries.length, exerciseCount, setCount, totalVolume };
  }, [history, periodDays]);

  const shiftCalendar = (direction) => {
    const next = new Date(calendarDate);
    if (calendarMode === "month") next.setMonth(next.getMonth() + direction);
    else next.setDate(next.getDate() + direction * 7);
    setCalendarDate(next);
  };

  const dayNames = ["月", "火", "水", "木", "金", "土", "日"];

  return (
    <div className="min-h-screen bg-zinc-950 text-zinc-100 p-3 sm:p-4 md:p-8">
      <div className="mx-auto max-w-7xl space-y-6">
        <div className="grid gap-6 xl:grid-cols-[1.05fr_0.95fr]">
          <div className="space-y-6">
            <Card className="rounded-3xl border-zinc-800 bg-zinc-900 text-zinc-100">
              <CardHeader className="pb-4 space-y-3">
                <CardTitle className="flex items-center gap-2 text-xl sm:text-2xl">
                  <Dumbbell className="h-5 w-5 sm:h-6 sm:w-6" />
                  セッション入力
                </CardTitle>
                <div className="rounded-2xl border border-zinc-800 bg-zinc-950 p-3 text-sm text-zinc-300">
                  自分のスマホ専用で使うなら、公開後にSafariで開いてホーム画面に追加するとかなりアプリっぽく使えます。
                </div>
              </CardHeader>
              <CardContent className="space-y-5">
                <div>
                  <Label>日付</Label>
                  <Input
                    type="date"
                    value={sessionDate}
                    onChange={(e) => setSessionDate(e.target.value)}
                    className="mt-2 h-11 border-zinc-700 bg-zinc-950 text-zinc-100 text-base"
                  />
                </div>

                <div className="space-y-3">
                  <div>
                    <Label>種目マスター追加</Label>
                    <Input
                      value={newExerciseName}
                      onChange={(e) => setNewExerciseName(e.target.value)}
                      placeholder="例: ポーズベンチ"
                      className="mt-2 h-11 border-zinc-700 bg-zinc-950 text-zinc-100 text-base"
                    />
                  </div>
                  <Button
                    onClick={addExerciseMaster}
                    variant="outline"
                    className="h-11 w-full rounded-2xl border-zinc-700 bg-zinc-950 text-zinc-100"
                  >
                    <Plus className="mr-2 h-4 w-4" />種目追加
                  </Button>
                </div>

                <div className="space-y-4">
                  {exercises.map((exercise) => (
                    <ExerciseInputCard
                      key={exercise.id}
                      exercise={exercise}
                      exerciseMaster={exerciseMaster}
                      updateExerciseName={updateExerciseName}
                      removeExerciseBlock={removeExerciseBlock}
                      addSetToExercise={addSetToExercise}
                      removeSetFromExercise={removeSetFromExercise}
                      updateSet={updateSet}
                    />
                  ))}
                </div>

                <Button
                  onClick={addExerciseBlock}
                  variant="outline"
                  className="h-11 w-full rounded-2xl border-zinc-700 bg-zinc-950 text-zinc-100"
                >
                  <Plus className="mr-2 h-4 w-4" />種目ブロック追加
                </Button>

                <div className="rounded-2xl border border-zinc-800 bg-zinc-950 px-4 py-3 text-sm text-zinc-300">
                  セッション総ボリューム: <span className="font-bold text-zinc-100">{sessionVolume} kg</span>
                </div>

                <div>
                  <Label>メモ</Label>
                  <Textarea
                    value={memo}
                    onChange={(e) => setMemo(e.target.value)}
                    placeholder="フォーム、違和感、疲労感、うまくいった点など"
                    className="mt-2 min-h-[110px] border-zinc-700 bg-zinc-950 text-zinc-100 text-base"
                  />
                </div>

                <div className="grid grid-cols-1 gap-3 sm:grid-cols-3">
                  <Button onClick={saveSession} className="h-11 rounded-2xl bg-white text-black hover:bg-zinc-200">
                    <Save className="mr-2 h-4 w-4" />保存
                  </Button>
                  <Button
                    onClick={exportImage}
                    className="h-11 rounded-2xl bg-red-600 text-white hover:bg-red-500"
                    disabled={isExporting}
                  >
                    <Download className="mr-2 h-4 w-4" />
                    {isExporting ? "画像生成中..." : "画像出力"}
                  </Button>
                  <Button
                    onClick={clearSession}
                    variant="outline"
                    className="h-11 rounded-2xl border-zinc-700 bg-zinc-950 text-zinc-100"
                  >
                    入力クリア
                  </Button>
                </div>

                <Button
                  onClick={() => setShowInstallGuide((prev) => !prev)}
                  variant="outline"
                  className="h-11 w-full rounded-2xl border-zinc-700 bg-zinc-950 text-zinc-100"
                >
                  <Smartphone className="mr-2 h-4 w-4" />
                  {showInstallGuide ? "ホーム画面追加の案内を閉じる" : "ホーム画面追加の案内を見る"}
                </Button>

                {showInstallGuide && (
                  <div className="rounded-2xl border border-zinc-800 bg-zinc-950 p-4 text-sm text-zinc-300 space-y-3">
                    <div className="font-semibold text-zinc-100">iPhoneでアプリっぽく使う手順</div>
                    <div>1. このツールをSafariで開く</div>
                    <div>2. 下の共有ボタン <Share2 className="inline h-4 w-4 align-[-2px]" /> を押す</div>
                    <div>3. 「ホーム画面に追加」を選ぶ</div>
                    <div>4. アイコン名を決めて追加する</div>
                    <div className="text-zinc-500">同じiPhone・同じブラウザなら記録はこの端末内に残ります。</div>
                  </div>
                )}
              </CardContent>
            </Card>

            <Card className="rounded-3xl border-zinc-800 bg-zinc-900 text-zinc-100">
              <CardHeader className="pb-4">
                <CardTitle className="flex items-center gap-2 text-xl sm:text-2xl">
                  <ClipboardList className="h-5 w-5 sm:h-6 sm:w-6" />
                  画像出力プレビュー
                </CardTitle>
              </CardHeader>
              <CardContent>
                <div className="overflow-auto rounded-3xl border border-zinc-800">
                  <div ref={captureRef} className="min-w-[320px] sm:min-w-[720px] md:min-w-[900px]">
                    <SessionSummaryCard session={currentSession} />
                  </div>
                </div>
              </CardContent>
            </Card>
          </div>

          <div className="space-y-6">
            <Card className="rounded-3xl border-zinc-800 bg-zinc-900 text-zinc-100">
              <CardHeader className="pb-4">
                <CardTitle className="flex items-center gap-2 text-xl sm:text-2xl">
                  <CalendarDays className="h-5 w-5 sm:h-6 sm:w-6" />
                  週・月カレンダー振り返り
                </CardTitle>
              </CardHeader>
              <CardContent className="space-y-5">
                <div className="space-y-3">
                  <div className="grid grid-cols-2 gap-2">
                    <Button className="rounded-2xl" variant={calendarMode === "week" ? "default" : "outline"} onClick={() => setCalendarMode("week")}>週表示</Button>
                    <Button className="rounded-2xl" variant={calendarMode === "month" ? "default" : "outline"} onClick={() => setCalendarMode("month")}>月表示</Button>
                  </div>
                  <div className="flex items-center justify-between gap-2">
                    <Button variant="outline" className="rounded-2xl border-zinc-700 bg-zinc-950 text-zinc-100" onClick={() => shiftCalendar(-1)}><ChevronLeft className="h-4 w-4" /></Button>
                    <div className="text-center text-sm font-semibold text-zinc-100">
                      {calendarMode === "month"
                        ? `${calendarDate.getFullYear()}年 ${calendarDate.getMonth() + 1}月`
                        : `${formatDateJP(periodDays[0])} 〜 ${formatDateJP(periodDays[6])}`}
                    </div>
                    <Button variant="outline" className="rounded-2xl border-zinc-700 bg-zinc-950 text-zinc-100" onClick={() => shiftCalendar(1)}><ChevronRight className="h-4 w-4" /></Button>
                  </div>
                </div>

                <div className="grid grid-cols-7 gap-1.5 sm:gap-2">
                  {dayNames.map((day) => (
                    <div key={day} className="rounded-xl bg-zinc-800 px-1 py-2 text-center text-[10px] font-semibold text-zinc-300 sm:px-2 sm:text-xs">{day}</div>
                  ))}
                  {periodDays.map((day) => {
                    const key = formatDateJP(day);
                    const items = entriesByDate.get(key) || [];
                    const vol = items.reduce((sum, item) => sum + numeric(item.totalVolume), 0);
                    const inMonth = day.getMonth() === calendarDate.getMonth();
                    return (
                      <div key={key} className={`min-h-[94px] rounded-2xl border p-1.5 sm:min-h-[112px] sm:p-2 ${inMonth || calendarMode === "week" ? "border-zinc-800 bg-zinc-950" : "border-zinc-900 bg-zinc-950/40 text-zinc-600"}`}>
                        <div className="mb-1 text-[10px] font-semibold sm:mb-2 sm:text-xs">{day.getDate()}日</div>
                        <div className="space-y-1 text-[9px] sm:text-[11px]">
                          <div className="text-zinc-400">記録: {items.length}</div>
                          <div className="text-zinc-400">Vol: {vol}</div>
                          {items.slice(0, 1).map((item) => (
                            <div key={item.id} className="rounded-lg bg-zinc-800 px-1.5 py-1 text-zinc-200 truncate">
                              {item.exercises[0]?.name || "セッション"}
                            </div>
                          ))}
                          {items.length > 1 && <div className="text-zinc-500">+{items.length - 1}件</div>}
                        </div>
                      </div>
                    );
                  })}
                </div>

                <div className="grid grid-cols-2 gap-3 sm:grid-cols-4">
                  <div className="rounded-2xl border border-zinc-800 bg-zinc-950 p-4"><div className="text-xs text-zinc-500">記録数</div><div className="mt-1 text-2xl font-bold">{periodSummary.entries}</div></div>
                  <div className="rounded-2xl border border-zinc-800 bg-zinc-950 p-4"><div className="text-xs text-zinc-500">種目数</div><div className="mt-1 text-2xl font-bold">{periodSummary.exerciseCount}</div></div>
                  <div className="rounded-2xl border border-zinc-800 bg-zinc-950 p-4"><div className="text-xs text-zinc-500">総セット数</div><div className="mt-1 text-2xl font-bold">{periodSummary.setCount}</div></div>
                  <div className="rounded-2xl border border-zinc-800 bg-zinc-950 p-4"><div className="text-xs text-zinc-500">総ボリューム</div><div className="mt-1 text-2xl font-bold">{periodSummary.totalVolume} kg</div></div>
                </div>
              </CardContent>
            </Card>

            <Card className="rounded-3xl border-zinc-800 bg-zinc-900 text-zinc-100">
              <CardHeader className="pb-4">
                <CardTitle className="flex items-center gap-2 text-xl sm:text-2xl">
                  <History className="h-5 w-5 sm:h-6 sm:w-6" />
                  保存履歴
                </CardTitle>
              </CardHeader>
              <CardContent>
                {history.length === 0 ? (
                  <div className="rounded-2xl border border-dashed border-zinc-700 p-8 text-center text-zinc-500">まだ保存されたセッションはありません。</div>
                ) : (
                  <div className="space-y-4">
                    {history.map((entry) => (
                      <div key={entry.id} className="rounded-2xl border border-zinc-800 bg-zinc-950 p-4">
                        <div className="mb-3 flex items-start justify-between gap-3">
                          <div>
                            <div className="font-semibold text-zinc-100">{displayDate(entry.date)}</div>
                            <div className="text-sm text-zinc-400">種目 {entry.exercises.length} / ボリューム {entry.totalVolume} kg</div>
                          </div>
                          <Button onClick={() => deleteSession(entry.id)} variant="ghost" className="rounded-2xl px-3 text-zinc-300"><Trash2 className="h-4 w-4" /></Button>
                        </div>

                        <div className="space-y-2 text-sm text-zinc-300">
                          {entry.exercises.map((exercise) => (
                            <div key={exercise.id} className="rounded-xl bg-zinc-900 p-3">
                              <div className="font-semibold text-zinc-100">{exercise.name}</div>
                              <div className="mt-1 text-zinc-400">{exercise.sets.length}セット / Vol {calcExerciseVolume(exercise)} kg</div>
                            </div>
                          ))}
                        </div>

                        {entry.memo && <div className="mt-3 rounded-xl bg-zinc-900 p-3 text-sm text-zinc-300 whitespace-pre-wrap">{entry.memo}</div>}

                        <Button
                          onClick={() => loadSession(entry)}
                          variant="outline"
                          className="mt-3 h-10 w-full rounded-2xl border-zinc-700 bg-zinc-900 text-zinc-100"
                        >
                          この内容を再読込
                        </Button>
                      </div>
                    ))}
                  </div>
                )}
              </CardContent>
            </Card>
          </div>
        </div>
      </div>
    </div>
  );
}
