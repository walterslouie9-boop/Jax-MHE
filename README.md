
JAX MHE
Inspection
import React, { useEffect, useMemo, useState } from "react";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Badge } from "@/components/ui/badge";
import { Tabs, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { CheckCircle2, AlertTriangle, Printer, RotateCcw, Save, Forklift, ClipboardCheck } from "lucide-react";
import { motion } from "framer-motion";

const EPJ_ITEMS = [
  "Leaks under PIT",
  "Drive wheels / platform wheels",
  "Platform condition",
  "Control arm",
  "Control handle",
  "Travel twist grip",
  "Quick Pick button",
  "Lower button",
  "Raise button",
  "Reversing button",
  "Grab bar",
  "High travel speed button",
  "Horn button",
  "Battery retainer plate",
  "Battery water level",
  "Vent caps",
  "Power disconnect",
  "Battery cables",
  "Load backrest",
  "Right fork",
  "Right fork wheel assembly",
  "Left fork wheel assembly",
  "Left fork",
  "Load rating & safety decals",
  "Horn operational check",
  "Raise forks",
  "Lower forks",
  "Travel forward / brake",
  "Reverse travel / brake",
  "Left turn",
  "Right turn",
  "Coast function",
  "Quick Pick function",
];

const FORKLIFT_TEMPLATE_ITEMS = [
  "No visible damage / structural defects",
  "No fluid leaks under truck",
  "Battery secure / charging cable condition",
  "Forks not bent, cracked, or damaged",
  "Load backrest secure",
  "Wheels / tires in good condition",
  "Capacity plate legible",
  "Safety decals readable",
  "Steering operates smoothly",
  "Horn operational",
  "Brake functions properly",
  "Lift / lower functions working",
  "Forward / reverse travel working",
  "Parking brake working",
  "Warning lights / alarms functioning",
  "Seat belt working (if equipped)",
];

const defaultMeta = () => ({
  operator: "",
  date: new Date().toISOString().slice(0, 10),
  equipmentId: "",
  hourMeter: "",
  shift: "",
  notes: "",
  supervisor: "",
});

function initChecklist(items) {
  return items.map((label) => ({ label, status: "unchecked", note: "" }));
}

export default function DailyInspectionApp() {
  const [equipmentType, setEquipmentType] = useState("EPJ");
  const [meta, setMeta] = useState(defaultMeta());
  const [epjChecklist, setEpjChecklist] = useState(initChecklist(EPJ_ITEMS));
  const [forkliftChecklist, setForkliftChecklist] = useState(initChecklist(FORKLIFT_TEMPLATE_ITEMS));
  const [showOnlyIssues, setShowOnlyIssues] = useState(false);

  const storageKey = useMemo(() => `inspection-app-${equipmentType.toLowerCase()}`, [equipmentType]);
  const checklist = equipmentType === "EPJ" ? epjChecklist : forkliftChecklist;
  const setChecklist = equipmentType === "EPJ" ? setEpjChecklist : setForkliftChecklist;

  useEffect(() => {
    const raw = localStorage.getItem(storageKey);
    if (!raw) return;
    try {
      const parsed = JSON.parse(raw);
      if (parsed.meta) setMeta(parsed.meta);
      if (Array.isArray(parsed.checklist)) setChecklist(parsed.checklist);
    } catch {}
  }, [storageKey]);

  useEffect(() => {
    localStorage.setItem(storageKey, JSON.stringify({ meta, checklist }));
  }, [storageKey, meta, checklist]);

  const counts = useMemo(() => {
    const pass = checklist.filter((i) => i.status === "pass").length;
    const fail = checklist.filter((i) => i.status === "fail").length;
    const na = checklist.filter((i) => i.status === "na").length;
    const unchecked = checklist.filter((i) => i.status === "unchecked").length;
    return { pass, fail, na, unchecked, total: checklist.length };
  }, [checklist]);

  const failedItems = checklist.filter((i) => i.status === "fail");
  const readyForService = failedItems.length === 0 && counts.unchecked === 0;

  const updateItem = (idx, patch) => {
    setChecklist((prev) => prev.map((item, i) => (i === idx ? { ...item, ...patch } : item)));
  };

  const resetCurrent = () => {
    setMeta(defaultMeta());
    setChecklist(initChecklist(equipmentType === "EPJ" ? EPJ_ITEMS : FORKLIFT_TEMPLATE_ITEMS));
    localStorage.removeItem(storageKey);
  };

  const markAll = (status) => {
    setChecklist((prev) => prev.map((item) => ({ ...item, status })));
  };

  const submitSummary = `
Equipment Type: ${equipmentType}
Operator: ${meta.operator || ""}
Date: ${meta.date || ""}
Equipment ID: ${meta.equipmentId || ""}
Hour Meter: ${meta.hourMeter || ""}
Shift: ${meta.shift || ""}
Supervisor: ${meta.supervisor || ""}
Passed: ${counts.pass}
Failed: ${counts.fail}
N/A: ${counts.na}
Unchecked: ${counts.unchecked}
Status: ${readyForService ? "SAFE TO OPERATE" : "REMOVE FROM SERVICE / REVIEW REQUIRED"}
Failed Items:
${failedItems.length ? failedItems.map((i) => `- ${i.label}${i.note ? `: ${i.note}` : ""}`).join("
") : "- None"}


Operator Notes:
${meta.notes || "None"}
`;


  const filteredChecklist = showOnlyIssues
    ? checklist.filter((item) => item.status === "fail" || item.note)
    : checklist;

  return (
    <div className="min-h-screen bg-slate-50 p-4 md:p-8">
      <div className="mx-auto max-w-7xl space-y-6">
        <motion.div
          initial={{ opacity: 0, y: 10 }}
          animate={{ opacity: 1, y: 0 }}
          className="flex flex-col gap-4 rounded-3xl bg-gradient-to-r from-red-700 to-red-500 p-6 text-white shadow-xl"
        >
          <div className="flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
            <div>
              <div className="mb-2 flex items-center gap-2">
                <Forklift className="h-5 w-5" />
                <Badge className="bg-white/20 text-white hover:bg-white/20">Daily Equipment Inspection</Badge>
              </div>
              <h1 className="text-2xl font-bold md:text-4xl">Forklift & EPJ Daily Check App</h1>
              <p className="mt-2 max-w-3xl text-sm text-red-50 md:text-base">
                Use this page to complete a pre-shift inspection, capture defects, and print or copy a summary for filing.
              </p>
            </div>
            <div className="grid grid-cols-2 gap-2 md:flex md:flex-wrap md:justify-end print:hidden">
              <Button variant="secondary" className="rounded-2xl" onClick={() => window.print()}>
                <Printer className="mr-2 h-4 w-4" /> Print
              </Button>
              <Button variant="secondary" className="rounded-2xl" onClick={() => navigator.clipboard.writeText(submitSummary)}>
                <ClipboardCheck className="mr-2 h-4 w-4" /> Copy Summary
              </Button>
              <Button variant="secondary" className="rounded-2xl" onClick={() => markAll("pass")}>
                <CheckCircle2 className="mr-2 h-4 w-4" /> Mark All Pass
              </Button>
              <Button variant="secondary" className="rounded-2xl" onClick={resetCurrent}>
                <RotateCcw className="mr-2 h-4 w-4" /> Reset
              </Button>
            </div>
          </div>
        </motion.div>

        <div className="grid gap-6 xl:grid-cols-[1.1fr_0.9fr]">
          <div className="space-y-6">
            <Card className="rounded-3xl shadow-sm">
              <CardHeader>
                <CardTitle>Inspection Setup</CardTitle>
                <CardDescription>Select equipment and enter operator details.</CardDescription>
              </CardHeader>
              <CardContent className="space-y-6">
                <Tabs value={equipmentType} onValueChange={setEquipmentType}>
                  <TabsList className="grid w-full grid-cols-2 rounded-2xl">
                    <TabsTrigger value="EPJ" className="rounded-2xl">EPJ</TabsTrigger>
                    <TabsTrigger value="Forklift" className="rounded-2xl">Forklift</TabsTrigger>
                  </TabsList>
                </Tabs>

                <div className="grid gap-4 md:grid-cols-2">
                  <div>
                    <label className="mb-2 block text-sm font-medium">Operator</label>
                    <Input value={meta.operator} onChange={(e) => setMeta((m) => ({ ...m, operator: e.target.value }))} placeholder="Enter operator name" className="rounded-2xl" />
                  </div>
                  <div>
                    <label className="mb-2 block text-sm font-medium">Date</label>
                    <Input type="date" value={meta.date} onChange={(e) => setMeta((m) => ({ ...m, date: e.target.value }))} className="rounded-2xl" />
                  </div>
                  <div>
                    <label className="mb-2 block text-sm font-medium">Equipment ID / Truck #</label>
                    <Input value={meta.equipmentId} onChange={(e) => setMeta((m) => ({ ...m, equipmentId: e.target.value }))} placeholder="e.g., EPJ-04" className="rounded-2xl" />
                  </div>
                  <div>
                    <label className="mb-2 block text-sm font-medium">Hour Meter</label>
                    <Input value={meta.hourMeter} onChange={(e) => setMeta((m) => ({ ...m, hourMeter: e.target.value }))} placeholder="Optional" className="rounded-2xl" />
                  </div>
                  <div>
                    <label className="mb-2 block text-sm font-medium">Shift</label>
                    <Input value={meta.shift} onChange={(e) => setMeta((m) => ({ ...m, shift: e.target.value }))} placeholder="1st / 2nd / 3rd" className="rounded-2xl" />
                  </div>
                  <div>
                    <label className="mb-2 block text-sm font-medium">Supervisor</label>
                    <Input value={meta.supervisor} onChange={(e) => setMeta((m) => ({ ...m, supervisor: e.target.value }))} placeholder="Optional" className="rounded-2xl" />
                  </div>
                </div>

                <div>
                  <label className="mb-2 block text-sm font-medium">Operator Notes</label>
                  <Textarea value={meta.notes} onChange={(e) => setMeta((m) => ({ ...m, notes: e.target.value }))} placeholder="Add any comments, issues, or follow-up actions" className="min-h-[90px] rounded-2xl" />
                </div>
              </CardContent>
            </Card>

            <Card className="rounded-3xl shadow-sm">
              <CardHeader>
                <div className="flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
                  <div>
                    <CardTitle>{equipmentType} Checklist</CardTitle>
                    <CardDescription>
                      {equipmentType === "EPJ"
                        ? "Detailed EPJ pre-use inspection items."
                        : "Suggested forklift template — align with the site posted electric truck checklist if your posted form differs."}
                    </CardDescription>
                  </div>
                  <Button variant="outline" className="rounded-2xl print:hidden" onClick={() => setShowOnlyIssues((v) => !v)}>
                    {showOnlyIssues ? "Show All Items" : "Show Issues Only"}
                  </Button>
                </div>
              </CardHeader>
              <CardContent>
                <div className="space-y-3">
                  {filteredChecklist.map((item, visibleIndex) => {
                    const idx = checklist.findIndex((i) => i.label === item.label);
                    const isFail = item.status === "fail";
                    const isPass = item.status === "pass";
                    return (
                      <motion.div
                        key={item.label}
                        initial={{ opacity: 0, y: 8 }}
                        animate={{ opacity: 1, y: 0 }}
                        transition={{ delay: visibleIndex * 0.01 }}
                        className={`rounded-2xl border p-4 ${isFail ? "border-amber-300 bg-amber-50" : isPass ? "border-emerald-200 bg-emerald-50" : "border-slate-200 bg-white"}`}
                      >
                        <div className="grid gap-3 lg:grid-cols-[1fr_auto] lg:items-start">
                          <div>
                            <div className="font-medium text-slate-900">{item.label}</div>
                            <Textarea
                              value={item.note}
                              onChange={(e) => updateItem(idx, { note: e.target.value })}
                              placeholder="Optional note"
                              className="mt-3 min-h-[70px] rounded-2xl bg-white"
                            />
                          </div>
                          <div className="flex flex-wrap gap-2 lg:flex-col lg:items-stretch">
                            <Button
                              className={`rounded-2xl ${isPass ? "bg-emerald-600 hover:bg-emerald-600" : ""}`}
                              variant={isPass ? "default" : "outline"}
                              onClick={() => updateItem(idx, { status: "pass" })}
                            >
                              Pass
                            </Button>
                            <Button
                              className={`rounded-2xl ${isFail ? "bg-amber-600 hover:bg-amber-600 text-white" : ""}`}
                              variant={isFail ? "default" : "outline"}
                              onClick={() => updateItem(idx, { status: "fail" })}
                            >
                              Fail
                            </Button>
                            <Button
                              className="rounded-2xl"
                              variant={item.status === "na" ? "default" : "outline"}
                              onClick={() => updateItem(idx, { status: "na" })}
                            >
                              N/A
                            </Button>
                            <Button
                              className="rounded-2xl"
                              variant={item.status === "unchecked" ? "default" : "ghost"}
                              onClick={() => updateItem(idx, { status: "unchecked" })}
                            >
                              Clear
                            </Button>
                          </div>
                        </div>
                      </motion.div>
                    );
                  })}
                </div>
              </CardContent>
            </Card>
          </div>

          <div className="space-y-6">
            <Card className="rounded-3xl shadow-sm">
              <CardHeader>
                <CardTitle>Inspection Status</CardTitle>
                <CardDescription>Quick view of completion and equipment disposition.</CardDescription>
              </CardHeader>
              <CardContent className="space-y-4">
                <div className="grid grid-cols-2 gap-3">
                  <Metric label="Pass" value={counts.pass} tone="emerald" />
                  <Metric label="Fail" value={counts.fail} tone="amber" />
                  <Metric label="N/A" value={counts.na} tone="slate" />
                  <Metric label="Unchecked" value={counts.unchecked} tone="blue" />
                </div>

                <div className={`rounded-3xl border p-4 ${readyForService ? "border-emerald-200 bg-emerald-50" : "border-red-200 bg-red-50"}`}>
                  <div className="flex items-center gap-3">
                    {readyForService ? <CheckCircle2 className="h-6 w-6 text-emerald-600" /> : <AlertTriangle className="h-6 w-6 text-red-600" />}
                    <div>
                      <div className="font-semibold text-slate-900">{readyForService ? "Safe to Operate" : "Remove From Service / Review Required"}</div>
                      <div className="text-sm text-slate-600">
                        {readyForService
                          ? "All checklist items are completed with no failed items."
                          : "Any failed item or unchecked item should be reviewed before operation."}
                      </div>
                    </div>
                  </div>
                </div>

                <div className="rounded-3xl border border-slate-200 bg-white p-4">
                  <div className="mb-3 text-sm font-semibold text-slate-900">Failed Items</div>
                  {failedItems.length ? (
                    <ul className="space-y-2 text-sm text-slate-700">
                      {failedItems.map((item) => (
                        <li key={item.label} className="rounded-2xl bg-slate-50 p-3">
                          <div className="font-medium">{item.label}</div>
                          {item.note ? <div className="mt-1 text-slate-600">{item.note}</div> : null}
                        </li>
                      ))}
                    </ul>
                  ) : (
                    <div className="text-sm text-slate-500">No failed items logged.</div>
                  )}
                </div>
              </CardContent>
            </Card>

            <Card className="rounded-3xl shadow-sm">
              <CardHeader>
                <CardTitle>Submission Summary</CardTitle>
                <CardDescription>Copy or print this section for daily records.</CardDescription>
              </CardHeader>
              <CardContent>
                <pre className="whitespace-pre-wrap rounded-2xl bg-slate-950 p-4 text-sm text-slate-100">{submitSummary}</pre>
              </CardContent>
            </Card>

            <Card className="rounded-3xl shadow-sm">
              <CardHeader>
                <CardTitle>Notes</CardTitle>
                <CardDescription>Implementation assumptions for this first version.</CardDescription>
              </CardHeader>
              <CardContent className="space-y-3 text-sm text-slate-700">
                <p>
                  EPJ items are based on the detailed pre-use inspection/toolbox talk reference you already have on file.
                </p>
                <p>
                  The forklift tab is a suggested electric truck template because your available sources require an electric truck daily check but do not include the full posted forklift checklist text in the search results.
                </p>
                <p>
                  This page stores entries locally in the browser for quick use during the shift. You can expand it later to match your exact posted form or add digital sign-off and compliance tracking.
                </p>
              </CardContent>
            </Card>
          </div>
        </div>
      </div>
    </div>
  );
}

function Metric({ label, value, tone = "slate" }) {
  const tones = {
    emerald: "bg-emerald-50 text-emerald-700 border-emerald-200",
    amber: "bg-amber-50 text-amber-700 border-amber-200",
    slate: "bg-slate-50 text-slate-700 border-slate-200",
    blue: "bg-blue-50 text-blue-700 border-blue-200",
  };

  return (
    <div className={`rounded-2xl border p-4 ${tones[tone] || tones.slate}`}>
      <div className="text-sm font-medium">{label}</div>
      <div className="mt-1 text-2xl font-bold">{value}</div>
    </div>
  );
}
# Jax-MHE
