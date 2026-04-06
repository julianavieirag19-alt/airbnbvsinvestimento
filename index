import { useState, useMemo } from "react";
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from "recharts";

const fmt = (v) => v.toLocaleString("pt-BR", { style: "currency", currency: "BRL", maximumFractionDigits: 0 });
const fmtPct = (v) => v.toFixed(2).replace(".", ",") + "%";

const IR_TABLE = [
  { dias: 180, aliq: 0.225 },
  { dias: 360, aliq: 0.20 },
  { dias: 720, aliq: 0.175 },
  { dias: Infinity, aliq: 0.15 },
];
function getIR(meses) {
  const dias = meses * 30;
  return IR_TABLE.find(r => dias <= r.dias).aliq;
}

const SLIDER = ({ label, value, min, max, step, onChange, display, hint }) => (
  <div style={{ marginBottom: 14 }}>
    <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 3 }}>
      <span style={{ fontSize: 13, color: "var(--color-text-secondary)" }}>{label}{hint && <span style={{ fontSize: 11, color: "var(--color-text-tertiary)", marginLeft: 5 }}>{hint}</span>}</span>
      <span style={{ fontSize: 13, fontWeight: 500 }}>{display}</span>
    </div>
    <input type="range" min={min} max={max} step={step} value={value} onChange={e => onChange(Number(e.target.value))} style={{ width: "100%" }} />
  </div>
);

const Card = ({ label, value, sub, color }) => (
  <div style={{ background: "var(--color-background-secondary)", borderRadius: "var(--border-radius-md)", padding: "0.75rem 1rem" }}>
    <div style={{ fontSize: 12, color: "var(--color-text-secondary)", marginBottom: 4 }}>{label}</div>
    <div style={{ fontSize: 20, fontWeight: 500, color: color || "var(--color-text-primary)" }}>{value}</div>
    {sub && <div style={{ fontSize: 11, color: "var(--color-text-tertiary)", marginTop: 2 }}>{sub}</div>}
  </div>
);

const CustomTooltip = ({ active, payload, label }) => {
  if (!active || !payload?.length) return null;
  const anos = Math.floor(label / 12), mr = label % 12;
  const tempo = anos > 0 ? `${anos}a${mr > 0 ? ` ${mr}m` : ""}` : `${label}m`;
  return (
    <div style={{ background: "var(--color-background-primary)", border: "0.5px solid var(--color-border-secondary)", borderRadius: 8, padding: "10px 14px", fontSize: 12 }}>
      <div style={{ fontWeight: 500, marginBottom: 6 }}>{tempo}</div>
      {payload.map(p => (
        <div key={p.dataKey} style={{ color: p.color, marginBottom: 2 }}>{p.name}: {fmt(p.value)}</div>
      ))}
    </div>
  );
};

export default function App() {
  const [valor, setValor] = useState(500000);
  const [diaria, setDiaria] = useState(350);
  const [ocupacao, setOcupacao] = useState(65);
  const [taxaAirbnb, setTaxaAirbnb] = useState(15);
  const [condominio, setCondominio] = useState(800);
  const [iptu, setIptu] = useState(200);
  const [manutencao, setManutencao] = useState(400);
  const [limpeza, setLimpeza] = useState(150);
  const [valorizacao, setValorizacao] = useState(4);
  const [cdi, setCdi] = useState(14.79);
  const [pctCDI, setPctCDI] = useState(110);
  const [horizonte, setHorizonte] = useState(60);

  const anos = Math.floor(horizonte / 12), mR = horizonte % 12;
  const tempoLabel = anos > 0 ? `${anos} ano${anos > 1 ? "s" : ""}${mR > 0 ? ` e ${mR} mês${mR > 1 ? "es" : ""}` : ""}` : `${horizonte} meses`;

  const calcMes = useMemo(() => {
    const diasOcupados = 30 * ocupacao / 100;
    const receitaBruta = diasOcupados * diaria;
    const taxaPlat = receitaBruta * taxaAirbnb / 100;
    const limpezaMes = diasOcupados > 0 ? Math.ceil(diasOcupados / 3) * limpeza : 0;
    const custos = taxaPlat + limpezaMes + condominio + iptu + manutencao;
    const ir = receitaBruta * 0.075; // carnê-leão simplificado ~7,5% médio
    const liquido = receitaBruta - custos - ir;
    return { receitaBruta, taxaPlat, limpezaMes, custos, ir, liquido };
  }, [diaria, ocupacao, taxaAirbnb, condominio, iptu, manutencao, limpeza]);

  const { chartData, airbnbFinal, invFinal } = useMemo(() => {
    const taxaAnual = cdi * pctCDI / 100;
    const taxaMensal = Math.pow(1 + taxaAnual / 100, 1 / 12) - 1;
    const valorizacaoMensal = Math.pow(1 + valorizacao / 100, 1 / 12) - 1;

    let airbnbAcum = 0;
    const data = [];

    for (let m = 0; m <= horizonte; m++) {
      const bruto = valor * Math.pow(1 + taxaMensal, m);
      const rend = bruto - valor;
      const ir = rend * getIR(m || 1);
      const invLiq = bruto - ir;

      const valorImovel = valor * Math.pow(1 + valorizacaoMensal, m);
      if (m > 0) airbnbAcum += calcMes.liquido;
      const airbnbTotal = airbnbAcum + valorImovel;

      data.push({ mes: m, investimento: Math.round(invLiq), airbnb: Math.round(airbnbTotal) });
    }

    return {
      chartData: data,
      airbnbFinal: data[horizonte].airbnb,
      invFinal: data[horizonte].investimento,
    };
  }, [valor, cdi, pctCDI, valorizacao, horizonte, calcMes]);

  const renda = calcMes.liquido;
  const yieldBruto = (calcMes.receitaBruta * 12 / valor * 100);
  const yieldLiquido = (renda * 12 / valor * 100);
  const taxaEfetiva = cdi * pctCDI / 100;
  const diff = airbnbFinal - invFinal;
  const melhor = diff > 0 ? "airbnb" : "investimento";

  return (
    <div style={{ padding: "1rem 0", maxWidth: 740, margin: "0 auto" }}>
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 24, marginBottom: 20 }}>

        <div>
          <div style={{ fontSize: 12, fontWeight: 500, color: "var(--color-text-tertiary)", marginBottom: 12, textTransform: "uppercase", letterSpacing: "0.06em" }}>Airbnb</div>
          <SLIDER label="Valor do imóvel" value={valor} min={100000} max={3000000} step={10000} onChange={setValor} display={fmt(valor)} />
          <SLIDER label="Diária média" value={diaria} min={50} max={2000} step={10} onChange={setDiaria} display={fmt(diaria)} />
          <SLIDER label="Ocupação" value={ocupacao} min={10} max={100} step={1} onChange={setOcupacao} display={`${ocupacao}%`} hint={`≈ ${Math.round(30 * ocupacao / 100)} dias/mês`} />
          <SLIDER label="Taxa Airbnb" value={taxaAirbnb} min={3} max={20} step={0.5} onChange={setTaxaAirbnb} display={`${taxaAirbnb}%`} />
          <SLIDER label="Condomínio" value={condominio} min={0} max={5000} step={50} onChange={setCondominio} display={fmt(condominio)} />
          <SLIDER label="IPTU (rateio mensal)" value={iptu} min={0} max={2000} step={10} onChange={setIptu} display={fmt(iptu)} />
          <SLIDER label="Manutenção / reserva" value={manutencao} min={0} max={3000} step={50} onChange={setManutencao} display={fmt(manutencao)} />
          <SLIDER label="Custo limpeza por saída" value={limpeza} min={0} max={500} step={10} onChange={setLimpeza} display={fmt(limpeza)} />
          <SLIDER label="Valorização anual do imóvel" value={valorizacao} min={0} max={15} step={0.5} onChange={setValorizacao} display={fmtPct(valorizacao)} />
        </div>

        <div>
          <div style={{ fontSize: 12, fontWeight: 500, color: "var(--color-text-tertiary)", marginBottom: 12, textTransform: "uppercase", letterSpacing: "0.06em" }}>Investimento (110% CDI)</div>
          <SLIDER label="CDI anual" value={cdi} min={5} max={25} step={0.01} onChange={setCdi} display={fmtPct(cdi)} />
          <SLIDER label="% do CDI" value={pctCDI} min={80} max={130} step={1} onChange={setPctCDI} display={`${pctCDI}%`} />
          <div style={{ padding: "10px 12px", background: "var(--color-background-info)", borderRadius: "var(--border-radius-md)", fontSize: 13, color: "var(--color-text-info)", marginBottom: 20 }}>
            Taxa efetiva: <strong>{fmtPct(taxaEfetiva)}</strong> ao ano
          </div>

          <div style={{ fontSize: 12, fontWeight: 500, color: "var(--color-text-tertiary)", marginBottom: 12, textTransform: "uppercase", letterSpacing: "0.06em" }}>Horizonte</div>
          <SLIDER label="Período de análise" value={horizonte} min={6} max={240} step={6} onChange={setHorizonte} display={tempoLabel} />

          <div style={{ marginTop: 16, borderTop: "0.5px solid var(--color-border-tertiary)", paddingTop: 16 }}>
            <div style={{ fontSize: 12, fontWeight: 500, color: "var(--color-text-tertiary)", marginBottom: 10, textTransform: "uppercase", letterSpacing: "0.06em" }}>Airbnb — resumo mensal</div>
            {[
              ["Receita bruta", fmt(calcMes.receitaBruta)],
              ["Taxa plataforma", `− ${fmt(calcMes.taxaPlat)}`],
              ["Limpeza", `− ${fmt(calcMes.limpezaMes)}`],
              ["Cond. + IPTU + manut.", `− ${fmt(condominio + iptu + manutencao)}`],
              ["IR estimado (carnê-leão)", `− ${fmt(calcMes.ir)}`],
            ].map(([k, v]) => (
              <div key={k} style={{ display: "flex", justifyContent: "space-between", fontSize: 12, color: "var(--color-text-secondary)", marginBottom: 5 }}>
                <span>{k}</span><span>{v}</span>
              </div>
            ))}
            <div style={{ display: "flex", justifyContent: "space-between", fontSize: 13, fontWeight: 500, borderTop: "0.5px solid var(--color-border-tertiary)", paddingTop: 8, marginTop: 4 }}>
              <span>Renda líquida/mês</span>
              <span style={{ color: renda >= 0 ? "var(--color-text-success)" : "var(--color-text-danger)" }}>{fmt(renda)}</span>
            </div>
          </div>
        </div>
      </div>

      <div style={{ borderTop: "0.5px solid var(--color-border-tertiary)", paddingTop: 18, marginBottom: 18 }}>
        <div style={{ fontSize: 12, fontWeight: 500, color: "var(--color-text-tertiary)", marginBottom: 12, textTransform: "uppercase", letterSpacing: "0.06em" }}>Resultado em {tempoLabel}</div>
        <div style={{ display: "grid", gridTemplateColumns: "repeat(2, minmax(0,1fr))", gap: 10, marginBottom: 10 }}>
          <Card label="Patrimônio via Airbnb (renda + imóvel)" value={fmt(airbnbFinal)} sub={`Yield líq. anual: ${fmtPct(yieldLiquido)} | Bruto: ${fmtPct(yieldBruto)}`} />
          <Card label="Patrimônio via investimento (líq. IR)" value={fmt(invFinal)} sub={`${fmtPct(taxaEfetiva)} a.a. — IR ${fmtPct(getIR(horizonte) * 100)}`} />
        </div>
        <div style={{ padding: "12px 16px", borderRadius: "var(--border-radius-md)", border: `1.5px solid ${diff > 0 ? "var(--color-border-success)" : "var(--color-border-danger)"}`, background: diff > 0 ? "var(--color-background-success)" : "var(--color-background-danger)" }}>
          <div style={{ fontSize: 13, color: diff > 0 ? "var(--color-text-success)" : "var(--color-text-danger)", fontWeight: 500 }}>
            {melhor === "airbnb"
              ? `Airbnb é mais vantajoso por ${fmt(diff)} neste cenário`
              : `Investimento é mais vantajoso por ${fmt(Math.abs(diff))} neste cenário`}
          </div>
          <div style={{ fontSize: 11, color: "var(--color-text-tertiary)", marginTop: 3 }}>
            Airbnb inclui renda acumulada + valorização estimada do imóvel ({fmtPct(valorizacao)} a.a.)
          </div>
        </div>
      </div>

      <div style={{ borderTop: "0.5px solid var(--color-border-tertiary)", paddingTop: 18 }}>
        <div style={{ fontSize: 12, fontWeight: 500, color: "var(--color-text-tertiary)", marginBottom: 10, textTransform: "uppercase", letterSpacing: "0.06em" }}>Evolução patrimonial</div>
        <div style={{ display: "flex", gap: 16, marginBottom: 10, fontSize: 12, color: "var(--color-text-secondary)" }}>
          <span style={{ display: "flex", alignItems: "center", gap: 4 }}><span style={{ width: 10, height: 10, borderRadius: 2, background: "#D85A30", display: "inline-block" }}></span> Airbnb (renda + imóvel)</span>
          <span style={{ display: "flex", alignItems: "center", gap: 4 }}><span style={{ width: 10, height: 10, borderRadius: 2, background: "#1D9E75", display: "inline-block" }}></span> Investimento (líq. IR)</span>
        </div>
        <div style={{ position: "relative", width: "100%", height: 240 }}>
          <ResponsiveContainer width="100%" height="100%">
            <LineChart data={chartData} margin={{ top: 4, right: 8, left: 0, bottom: 0 }}>
              <CartesianGrid strokeDasharray="3 3" stroke="rgba(128,128,128,0.15)" />
              <XAxis dataKey="mes" tickFormatter={m => m % 24 === 0 ? `${m / 12}a` : ""} tick={{ fontSize: 11 }} />
              <YAxis tickFormatter={v => `${(v / 1000).toFixed(0)}k`} tick={{ fontSize: 11 }} width={52} />
              <Tooltip content={<CustomTooltip />} />
              <Line type="monotone" dataKey="airbnb" name="Airbnb" stroke="#D85A30" dot={false} strokeWidth={2} />
              <Line type="monotone" dataKey="investimento" name="Investimento" stroke="#1D9E75" dot={false} strokeWidth={2} />
            </LineChart>
          </ResponsiveContainer>
        </div>
        <div style={{ fontSize: 11, color: "var(--color-text-tertiary)", marginTop: 8 }}>
          * IR do investimento pelas alíquotas regressivas (22,5% → 15%). IR do Airbnb estimado em 7,5% sobre receita bruta (carnê-leão — alíquota real depende da sua faixa de renda). Não considera vacância prolongada, custos de mobília, reformas ou gestão por terceiros.
        </div>
      </div>
    </div>
  );
}
