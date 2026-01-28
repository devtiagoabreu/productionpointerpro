# **Documentação Completa - Production Pointer Pro**

## **Índice**
1. [Visão Geral do Sistema](#1-visão-geral-do-sistema)
2. [Arquitetura do Sistema](#2-arquitetura-do-sistema)
3. [Fluxos de Trabalho](#3-fluxos-de-trabalho)
4. [Modelo de Dados](#4-modelo-de-dados)
5. [Metodologia LEAN Integrada](#5-metodologia-lean-integrada)
6. [KPIs e Métricas](#6-kpis-e-métricas)
7. [Interface do Usuário](#7-interface-do-usuário)
8. [Integrações](#8-integrações)
9. [Futuras Expansões](#9-futuras-expansões)

---

## **1. Visão Geral do Sistema**

### **1.1 Objetivo Principal**
Sistema de apontamento de produção para indústria têxtil com foco na metodologia LEAN, integrado ao ERP Systêxtil, que permite:
- Apontamento em tempo real via dispositivos móveis
- Controle de produção por máquina e operador
- Cálculo automático de eficiências e KPIs
- Gestão visual do chão de fábrica
- Previsão automática de produção baseada em velocidades específicas por produto

### **1.2 Público-Alvo**
- **Operadores**: Apontamento diário de produção
- **Supervisores**: Monitoramento em tempo real
- **Planejadores**: Controle de capacidade e programação
- **Manutenção**: Registro de paradas e intervenções
- **Gerência**: Análise de indicadores e tomada de decisão

### **1.3 Benefícios Esperados**
- Redução de 30% no tempo de apontamento
- Aumento de 15% na eficiência operacional
- Redução de 25% no WIP (Work in Progress)
- Eliminação de planilhas manuais
- Decisões baseadas em dados em tempo real

---

## **2. Arquitetura do Sistema**

### **2.1 Diagrama de Arquitetura**
```
┌─────────────────────────────────────────────────────────────┐
│                        DISPOSITIVOS                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Mobile  │  │  Tablet  │  │ Desktop  │  │  ESP32   │   │
│  │ (Celular)│  │          │  │          │  │(Automação)│   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │              │             │             │          │
└───────┼──────────────┼─────────────┼─────────────┼──────────┘
        │              │             │             │
        └──────────────┼─────────────┼─────────────┘
                       │             │
                ┌──────▼─────────────▼──────┐
                │        API REST Flask      │
                │     (Python + JWT Auth)    │
                └─────────────┬──────────────┘
                              │
                ┌─────────────▼──────────────┐
                │        Banco de Dados       │
                │    (PostgreSQL + Redis)     │
                └─────────────┬──────────────┘
                              │
                ┌─────────────▼──────────────┐
                │   ERP Systêxtil (API)      │
                │  + Sistemas Legados        │
                └─────────────────────────────┘
```

### **2.2 Stack Tecnológica**
- **Backend**: Python Flask, SQLAlchemy, Celery
- **Frontend**: HTML5, CSS3, JavaScript (Vanilla + Vue.js)
- **Banco de Dados**: PostgreSQL (dados transacionais), Redis (cache e filas)
- **Autenticação**: JWT com refresh tokens
- **QR Code**: ZXing (leitura), qrcode (geração)
- **WebSockets**: Socket.IO para atualizações em tempo real
- **Containerização**: Docker + Docker Compose
- **CI/CD**: GitHub Actions

### **2.3 Componentes Principais**
1. **Módulo de Autenticação**: Controle de acesso multi-nível
2. **Módulo de Apontamento**: Fluxo principal de produção
3. **Módulo de Administração**: Cadastros e configurações
4. **Módulo de Relatórios**: KPIs e análises
5. **Módulo de Integração**: API com ERP e futuros sistemas
6. **Módulo de Notificações**: Alertas em tempo real

---

## **3. Fluxos de Trabalho**

### **3.1 Fluxo Principal - Apontamento de Produção**
```
┌─────────────────┐
│   Login Operador│
└────────┬────────┘
         │
┌────────▼────────┐
│ Escanear Máquina│
└────────┬────────┘
         │
    ┌────▼────┐
    │Máquina  │
    │ Parada? │
    └────┬────┘
         ├─────────SIM─────────┐
    ┌────▼────┐          ┌─────▼─────┐
    │Tem OP   │          │Sem OP     │
    │Associada?│          │Associada  │
    └────┬────┘          └─────┬─────┘
         ├─────SIM─────┐       │
    ┌────▼────┐   ┌────▼────┐ │
    │Mostrar  │   │Oferecer │ │
    │OP Atual │   │Opções   │ │
    └────┬────┘   └────┬────┘ │
         │             │      │
         └─────┐ ┌─────┘      │
               │ │           │
          ┌────▼─▼───────────▼────┐
          │Retomar │ Registrar  │ Nova  │
          │Produção│ Parada     │ OP    │
          └────────┴──────────────┘
```

### **3.2 Fluxo Detalhado - Máquina Parada COM OP**
```
Operador escaneia máquina parada → Sistema detecta OP associada → Mostra:
1. Dados da OP (número, produto, metragem)
2. Status atual (aguardando material, manutenção, etc.)
3. Tempo parado
4. Opções:
   - ▶ Retomar produção
   - ⏸ Registrar novo motivo de parada
   - 🔄 Transferir para outra máquina
   - ✗ Finalizar associação (OP concluída)
   - 📋 Adicionar observação
```

### **3.3 Fluxo Detalhado - Máquina Parada SEM OP**
```
Operador escaneia máquina parada → Sistema não encontra OP → Mostra:
1. Status: "Máquina disponível"
2. Opções:
   - 📷 Escanear nova OP
   - 🔧 Registrar manutenção preventiva
   - ⚙ Realizar setup/montagem
   - 🧹 Limpeza geral
   - 📋 Registrar observação geral
```

### **3.4 Fluxo - Início de Nova Produção**
```
1. Escanear máquina (deve estar parada)
2. Escanear QR Code da OP
3. Sistema verifica:
   - OP está liberada para produção?
   - Máquina é compatível com etapa atual?
   - Há restrições? (material, qualidade, etc.)
4. Confirmar dados da OP:
   - Metragem programada: 5.000m
   - Produto: 1.23456.789.123456
   - Velocidade padrão: 15 m/min (para tingir)
   - Tempo estimado: 5h33min + 60min setup
5. Operador confirma ou ajusta metragem inicial
6. Sistema inicia cronômetro
```

### **3.5 Fluxo - Finalização de Produção**
```
1. Operador escaneia OP em produção
2. Sistema mostra:
   - Metragem produzida (calculada automaticamente)
   - Tempo total de produção
   - Eficiência calculada em tempo real
3. Operador pode:
   - ✅ Confirmar metragem (pré-preenchida)
   - ✏️ Ajustar metragem (com justificativa)
   - 📝 Adicionar observações
   - 🏷️ Registrar problemas de qualidade
4. Sistema:
   - Atualiza status da OP
   - Libera máquina
   - Calcula KPIs
   - Envia notificação para supervisor
```

### **3.6 Fluxo - Registro de Parada**
```
Durante produção → Operador seleciona "Registrar Parada" → Sistema:
1. Para cronômetro de produção
2. Mostra categorias de parada:
   ├── 🛠️ Manutenção
   │   ├── Corretiva
   │   ├── Preventiva
   │   └── Preditiva
   ├── ⚡ Energia/Utilidades
   ├── 📦 Material
   │   ├── Falta de matéria-prima
   │   ├── Aguardando OP anterior
   │   └── Problema qualidade material
   ├── 👷‍♂️ Operacional
   │   ├── Troca de turno
   │   ├── Almoço/intervalo
   │   └── Reunião/treinamento
   └── 🧪 Controle Qualidade
       ├── Amostragem
       ├── Ajuste processo
       └── Aguardando liberação
3. Operador seleciona motivo específico
4. Adiciona observações (opcional)
5. Tira foto (opcional)
6. Sistema inicia cronômetro de parada
```

---

## **4. Modelo de Dados**

### **4.1 Diagrama Entidade-Relacionamento (ER)**
```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│     USUÁRIOS    │      │    MÁQUINAS     │      │    PRODUTOS     │
├─────────────────┤      ├─────────────────┤      ├─────────────────┤
│ • id            │      │ • id            │      │ • id            │
│ • username      │◄─────│ • code          │      │ • product_code  │
│ • password_hash │      │ • name          │      │ • description   │
│ • name          │      │ • type          │      │ • level         │
│ • user_type     │      │ • qr_code       │      │ • group         │
│ • department    │      │ • status        │      │ • subgroup      │
│ • is_active     │      │ • department    │      │ • item          │
└────────┬────────┘      │ • last_maintenance│    │ • fabric_type   │
         │               └────────┬────────┘      │ • fabric_weight │
         │                        │               │ • fabric_width  │
         │               ┌────────▼────────┐      │ • color_type    │
         │               │ ASSOCIAÇÃO OP   │      └────────┬────────┘
         │               ├─────────────────┤               │
         │               │ • id            │               │
┌────────▼────────┐      │ • machine_id    │◄──────────────┘
│   APONTAMENTOS  │      │ • op_id         │
├─────────────────┤      │ • type          │      ┌─────────────────┐
│ • id            │      │ • start_time    │      │  VELOCIDADES    │
│ • user_id       ├──────│ • end_time      │      ├─────────────────┤
│ • machine_id    │      │ • is_active     │      │ • id            │
│ • op_id         │      └────────┬────────┘      │ • product_code  │
│ • start_time    │               │               │ • stage         │
│ • end_time      │               │               │ • machine_type  │
│ • produced_meters│      ┌────────▼────────┐      │ • standard_speed│
│ • expected_meters│      │   OPs - ERP     │      │ • min_speed     │
│ • observation   │      ├─────────────────┤      │ • max_speed     │
│ • status        │      │ • id            │      │ • setup_time    │
└─────────────────┘      │ • op_number     ├──────│ • target_eff    │
         │               │ • product_code  │      └─────────────────┘
         │               │ • programmed_m  │               │
         │               │ • loaded_m      │               │
┌────────▼────────┐      │ • produced_m    │      ┌────────▼────────┐
│     PARADAS     │      │ • status        │      │    ROTEIROS     │
├─────────────────┤      │ • qr_code       │      ├─────────────────┤
│ • id            │      └─────────────────┘      │ • id            │
│ • machine_id    │               │               │ • product_code  │
│ • stop_reason_id│               │               │ • stage         │
│ • start_time    │      ┌────────▼────────┐      │ • order         │
│ • end_time      │      │ MOTIVO PARADA   │      │ • machine_type  │
│ • observation   │      ├─────────────────┤      │ • next_stage    │
│ • user_id       │      │ • id            │      └─────────────────┘
└─────────────────┘      │ • code          │
                         │ • description   │
                         │ • category      │
                         │ • is_planned    │
                         └─────────────────┘
```

### **4.2 Tabelas Detalhadas**

#### **4.2.1 Produtos e Velocidades**
```sql
-- Hierarquia completa do produto
Products {
  product_code: "1.23456.789.123456"  -- nível.grupo.subgrupo.item
  description: "Tecido Algodão 200g/m²"
  level: "1"              -- Nível hierárquico
  group: "23456"          -- Grupo (ex: algodão)
  subgroup: "789"         -- Subgrupo (ex: cru)
  item: "123456"          -- Item específico
  
  -- Características técnicas
  fabric_type: "algodão"
  fabric_weight: 200.00   -- g/m²
  fabric_width: 150.00    -- cm
  color_type: "liso"      -- liso, estampado, listrado
  
  -- Roteiro de produção (JSON)
  production_route: [
    {"stage": "preparar", "machine_type": "urdideira", "order": 1},
    {"stage": "tingir", "machine_type": "jigger", "order": 2},
    {"stage": "secar", "machine_type": "secadeira", "order": 3},
    {"stage": "estampar", "machine_type": "stork", "order": 4, "optional": true},
    {"stage": "acabar", "machine_type": "calandra", "order": 5},
    {"stage": "revisar", "machine_type": "revisadeira", "order": 6}
  ]
}

-- Velocidades específicas por produto e etapa
ProductSpeeds {
  product_code: "1.23456.789.123456"
  production_stage: "tingir"
  machine_type: "jigger"
  
  -- Velocidades (metros por minuto)
  standard_speed: 15.00   -- Padrão para este produto
  min_speed: 12.00        -- Mínimo permitido
  max_speed: 18.00        -- Máximo permitido
  
  -- Tempos auxiliares (minutos)
  setup_time: 60          -- Tempo preparação
  cleaning_time: 30       -- Tempo limpeza pós-produção
  
  -- Metas
  target_efficiency: 92.00  -- Eficiência alvo %
  quality_standard: 98.50   -- Qualidade mínima aceitável %
  
  -- Fatores de ajuste
  color_factor_dark: 0.80   -- Para cores escuras (reduz velocidade)
  color_factor_light: 1.00  -- Para cores claras
  complexity_factor_high: 0.70  -- Para estampas complexas
}
```

#### **4.2.2 Associação OP-Máquina (Status em Tempo Real)**
```sql
MachineOPAssociation {
  machine_id: 15          -- Máquina específica
  op_id: 12345            -- OP específica
  association_type: "waiting"  -- production, waiting, setup, maintenance
  start_time: "2024-01-15 08:30:00"
  end_time: NULL          -- Ainda ativa
  is_active: TRUE
  
  -- Dados calculados em tempo real
  calculated_duration: 125  -- minutos desde start_time
  expected_completion: "2024-01-15 16:45:00"
  efficiency_so_far: 87.3   -- % até o momento
}
```

#### **4.2.3 Mapa de Produção (View em Tempo Real)**
```sql
CREATE VIEW production_map_live AS
SELECT 
    m.name as machine_name,
    m.status as machine_status,
    po.op_number,
    po.product_code,
    p.description as product_description,
    moa.association_type,
    EXTRACT(EPOCH FROM (NOW() - moa.start_time))/60 as duration_minutes,
    u.name as operator_name,
    
    -- Cálculos baseados na velocidade do produto
    ROUND(
        (EXTRACT(EPOCH FROM (NOW() - moa.start_time))/60 - ps.setup_time) 
        * ps.standard_speed_m_min, 2
    ) as estimated_meters_produced
    
FROM machines m
LEFT JOIN machine_op_association moa ON m.id = moa.machine_id AND moa.is_active = TRUE
LEFT JOIN production_orders po ON moa.op_id = po.id
LEFT JOIN products p ON po.product_code = p.product_code
LEFT JOIN product_speeds ps ON p.product_code = ps.product_code 
    AND ps.production_stage = CURRENT_STAGE_FUNCTION(po.id)
LEFT JOIN users u ON moa.associated_by = u.id
ORDER BY m.department, m.name;
```

---

## **5. Metodologia LEAN Integrada**

### **5.1 Ferramentas LEAN Implementadas**

#### **5.1.1 Andon Visual (Painel de Produção)**
```
┌─────────────────────────────────────────────────────────────────┐
│                    PAINEL ANDON - TINGIMENTO                    │
├─────┬────────────┬──────────┬─────────┬──────────┬─────────────┤
│ Máq │    OP      │ Status   │ Tempo   │ Efic.    │ Observações │
├─────┼────────────┼──────────┼─────────┼──────────┼─────────────┤
│ J01 │ OP-12345   │ 🟢 Produz│ 2h15m   │ 92%      │ Normal      │
│ J02 │ OP-12346   │ 🟡 Parada│ 45m     │ -        │ Falta tinta │
│ J03 │ OP-12347   │ 🔴 Manut.│ 3h10m   │ -        │ Vazamento   │
│ J04 │ -          │ ⚪ Livre │ -       │ -        │ Disponível  │
└─────┴────────────┴──────────┴─────────┴──────────┴─────────────┘
```

#### **5.1.2 Kanban Digital**
- **Cartões virtuais** por OP
- **Fluxo visual** entre máquinas
- **Limites de WIP** por departamento
- **Sinalização pull** quando máquina disponível

#### **5.1.3 5S Digital**
1. **Seiri (Organizar)**: Controle de ferramentas e materiais
2. **Seiton (Ordenar)**: Layout virtual das máquinas
3. **Seiso (Limpeza)**: Registro de limpezas programadas
4. **Seiketsu (Padronizar)**: Procedimentos digitais
5. **Shitsuke (Disciplina)**: Acompanhamento de conformidade

### **5.2 Fluxo de Valor (Value Stream Mapping)**
```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  Programação │   │  Preparação │   │  Produção   │   │   Qualidade │
│     ERP      │──▶│   Setup     │──▶│   Máquina   │──▶│   Inspeção  │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
       │                 │                 │                 │
       ▼                 ▼                 ▼                 ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ Lead Time:  │   │   Setup:    │   │  Processo:  │   │   Aguardo:  │
│    1 dia    │   │   60 min    │   │   5h30min   │   │   30 min    │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
```

### **5.3 SMED (Single Minute Exchange of Die)**
- **Setup interno** × **Setup externo** registrados
- **Tempos padrão** por tipo de máquina e produto
- **Análise de setup** para redução de tempo
- **Checklists digitais** para troca de produto

---

## **6. KPIs e Métricas**

### **6.1 KPIs Principais**

#### **6.1.1 OEE (Overall Equipment Effectiveness)**
```
OEE = Disponibilidade × Desempenho × Qualidade

• Disponibilidade = (Tempo Produção / Tempo Disponível) × 100
• Desempenho = (Produção Real / Produção Teórica) × 100
• Qualidade = (Peças Boas / Total Produzido) × 100

Exemplo:
Tempo Disponível: 480 min (8h)
Tempo Produção: 390 min
Produção Teórica: 5.000m (390 min × 15 m/min)
Produção Real: 4.200m
Peças Boas: 4.116m (98% qualidade)

Disponibilidade = (390/480) × 100 = 81.25%
Desempenho = (4.200/5.000) × 100 = 84.00%
Qualidade = (4.116/4.200) × 100 = 98.00%

OEE = 0.8125 × 0.84 × 0.98 = 66.86%
```

#### **6.1.2 Eficiência por Produto**
```
Eficiência Produto = (Velocidade Real / Velocidade Padrão) × 100

Produto A: Padrão 15 m/min, Real 14.2 m/min → 94.7%
Produto B: Padrão 10 m/min, Real 9.1 m/min → 91.0%
Produto C: Padrão 8 m/min, Real 7.0 m/min → 87.5%
```

#### **6.1.3 Quebra/Taxa de Rejeição**
```
Quebra = (Metragem Carregada - Metragem Boa) / Metragem Carregada × 100

OP-12345: Carregado 5.000m, Bom 4.900m → Quebra 2.00%
OP-12346: Carregado 3.000m, Bom 2.850m → Quebra 5.00%
```

### **6.2 Dashboard de Métricas**
```
┌─────────────────────────────────────────────────────────────┐
│                 DASHBOARD PRODUÇÃO - HOJE                    │
├─────────────────┬─────────────────┬─────────────────┬───────┤
│      OEE        │   EFICIÊNCIA    │     QUEBRA      │  WIP  │
│     67.2%       │     89.3%       │      3.2%       │ 8 OPs │
├─────────────────┼─────────────────┼─────────────────┼───────┤
│  TEMPO PROD.    │   TEMPO PARADA  │   DISPONIBIL.   │ MTBF  │
│    412 min      │     68 min      │     85.8%       │ 32h   │
├─────────────────┴─────────────────┴─────────────────┴───────┤
│                    TOP 5 MOTIVOS PARADA                     │
├─────┬──────────────────────────────┬──────────┬─────────────┤
│ #   │          MOTIVO              │  TEMPO   │     %       │
├─────┼──────────────────────────────┼──────────┼─────────────┤
│ 1   │ Falta Material               │  125 min │    18.4%    │
│ 2   │ Manutenção Corretiva         │   98 min │    14.4%    │
│ 3   │ Troca de Produto/Setup       │   87 min │    12.8%    │
│ 4   │ Aguardando Liberação Qualid. │   65 min │     9.6%    │
│ 5   │ Reunião/Treinamento          │   42 min │     6.2%    │
└─────┴──────────────────────────────┴──────────┴─────────────┘
```

### **6.3 Relatórios Automáticos**

#### **6.3.1 Relatório Diário de Produção**
- Resumo por turno
- Comparativo com meta
- Principais desvios
- Ações corretivas necessárias

#### **6.3.2 Relatório Semanal de Performance**
- Evolução dos KPIs
- Análise de tendências
- Ranking de máquinas/operadores
- Planejamento da semana seguinte

#### **6.3.3 Relatório Mensal Lean**
- Perdas mapeadas (7 desperdícios)
- Propostas de melhoria
- ROI de ações implementadas
- Metas para próximo mês

---

## **7. Interface do Usuário**

### **7.1 Design Principles**
- **Mobile First**: Interface otimizada para celular
- **One Hand Use**: Botões grandes e acessíveis
- **Scan First**: Minimizar digitação
- **Visual Feedback**: Confirmações imediatas
- **Offline Support**: Funcionalidade básica sem internet

### **7.2 Telas Principais**

#### **7.2.1 Tela de Login (Mobile)**
```
┌───────────────────────────────────┐
│        PRODUCTION POINTER         │
│             Version 1.0           │
├───────────────────────────────────┤
│  [Ícone Usuário]                  │
│  ┌─────────────────────────┐      │
│  │  Usuário:               │      │
│  │  ┌─────────────────┐    │      │
│  │  │                 │    │      │
│  │  └─────────────────┘    │      │
│  │                         │      │
│  │  Senha:                 │      │
│  │  ┌─────────────────┐    │      │
│  │  │ •••••••••       │    │      │
│  │  └─────────────────┘    │      │
│  │                         │      │
│  │  [   ENTRAR   ]         │      │
│  └─────────────────────────┘      │
│                                   │
│  Problemas para acessar?          │
│  Contate o supervisor.            │
└───────────────────────────────────┘
```

#### **7.2.2 Dashboard Operador**
```
┌───────────────────────────────────┐
│  👤 João Silva • Turno: Manhã     │
├───────────────────────────────────┤
│                                    │
│  [ 📷 ESCANEAR QR CODE ]          │
│                                    │
│  ┌─────────────────────────┐      │
│  │ PRODUÇÃO ATUAL          │      │
│  │ Máquina: Jigger 01      │      │
│  │ OP: OP-12345            │      │
│  │ Produto: Algodão 200g   │      │
│  │ Tempo: 2h15m            │      │
│  │ Metros: 1.950/5.000     │      │
│  │ Eficiência: 92%         │      │
│  │                         │      │
│  │ [ ADIC. OBSERVAÇÃO ]    │      │
│  │ [ REGISTRAR PARADA ]    │      │
│  │ [ FINALIZAR ]           │      │
│  └─────────────────────────┘      │
│                                    │
│  HISTÓRICO HOJE (3 apontamentos)   │
│  • OP-12344: 1.200m @ 89%          │
│  • OP-12343: 800m @ 94%            │
│  • OP-12342: 1.500m @ 91%          │
│                                    │
└───────────────────────────────────┘
```

#### **7.2.3 Tela de Apontamento Ativo**
```
┌───────────────────────────────────┐
│  ⏸ PRODUÇÃO PAUSADA               │
├───────────────────────────────────┤
│                                    │
│  MÁQUINA: Jigger 02               │
│  OP: OP-12346                     │
│  MOTIVO PARADA: Falta de Tinta    │
│  TEMPO PARADO: 45 minutos         │
│                                    │
│  ┌─────────────────────────┐      │
│  │                         │      │
│  │  O que deseja fazer?    │      │
│  │                         │      │
│  │  [ ▶ RETOMAR ]          │      │
│  │  [ ✏ EDITAR MOTIVO ]    │      │
│  │  [ 📷 ADIC. FOTO ]      │      │
│  │  [ ❌ CANCELAR OP ]     │      │
│  │                         │      │
│  └─────────────────────────┘      │
│                                    │
│  Supervisor notificado             │
│  Estoque: Solicitar tinta azul     │
│                                    │
└───────────────────────────────────┘
```

#### **7.2.4 Painel Andon (Tablet/Desktop)**
```
┌─────────────────────────────────────────────────────────────────┐
│  🏭 PAINEL DE PRODUÇÃO • TINGIMENTO • 15/01/2024 14:30          │
├─────┬────────────┬──────────┬─────────┬──────────┬─────────────┤
│     │            │          │         │          │             │
│ MÁQ │    OP      │  STATUS  │  TEMPO  │  EFIC.   │  PRÓXIMA AÇÃO│
├─────┼────────────┼──────────┼─────────┼──────────┼─────────────┤
│     │            │          │         │          │             │
│ J01 │ OP-12345   │ 🟢 92%   │ 2h15m   │  94.2%   │ Continuar   │
│     │ Algodão    │          │         │          │             │
├─────┼────────────┼──────────┼─────────┼──────────┼─────────────┤
│     │            │          │         │          │             │
│ J02 │ OP-12346   │ 🟡 45min │ 45min   │   --     │ Aguard. Tin.│
│     │ Poliéster  │          │         │          │             │
├─────┼────────────┼──────────┼─────────┼──────────┼─────────────┤
│     │            │          │         │          │             │
│ J03 │ OP-12347   │ 🟢 88%   │ 1h30m   │  91.5%   │ Continuar   │
│     │ Viscose    │          │         │          │             │
├─────┼────────────┼──────────┼─────────┼──────────┼─────────────┤
│     │            │          │         │          │             │
│ J04 │ --         │ ⚪ Livre │ --      │   --     │ Próx. OP    │
│     │            │          │         │          │             │
└─────┴────────────┴──────────┴─────────┴──────────┴─────────────┘
│  ESTATÍSTICAS: OEE 67% • PROD. 8.150m • QUEBRA 2.3% • WIP 6 OPs │
└─────────────────────────────────────────────────────────────────┘
```

### **7.3 Navegação por QR Code**
```
┌───────────────────────────────────┐
│          SISTEMA DE SCAN          │
│                                    │
│  ┌─────────────────────────┐      │
│  │                         │      │
│  │     [CAMERA ATIVA]      │      │
│  │                         │      │
│  │  Centralize o QR Code   │      │
│  │  na área destacada      │      │
│  │                         │      │
│  └─────────────────────────┘      │
│                                    │
│  O que deseja escanear?            │
│                                    │
│  [  MÁQUINA  ]  [   OP    ]       │
│                                    │
│  Últimos escaneamentos:            │
│  14:25 - Jigger 01 (OP-12345)      │
│  14:10 - OP-12346 (Poliéster)      │
│                                    │
└───────────────────────────────────┘
```

---

## **8. Integrações**

### **8.1 Integração com ERP Systêxtil**
```
Fluxo de Sincronização:
1. Sistema busca OPs novas/alteradas via API do ERP
2. Valida dados e carrega no banco local
3. Atualiza status das OPs em produção
4. Envia apontamentos concluídos para o ERP
5. Sincroniza estoque de materiais

Endpoints ERP:
GET /api/erp/production-orders     → Listar OPs
GET /api/erp/products/{code}       → Detalhes produto
POST /api/erp/production-pointing  → Enviar apontamentos
GET /api/erp/material-stock        → Consultar estoque
```

### **8.2 Futura Integração com ESP32**
```
Componentes do ESP32:
• Sensor de velocidade (encoder)
• Medidor de metros (sensor óptico)
• Contador de paradas (sensor de movimento)
• Sensor de temperatura (para secagem)
• Conexão Wi-Fi/Bluetooth

Fluxo de Dados:
ESP32 → Coleta dados da máquina
     → Envia via MQTT/HTTP
     → Sistema processa e atualiza em tempo real
     → Alerta se fora dos parâmetros
```

### **8.3 API para Sistemas Terceiros**
```python
# Endpoints disponíveis
POST /api/v1/production/start      # Iniciar produção
POST /api/v1/production/stop       # Parar produção
GET  /api/v1/production/status     # Status atual
GET  /api/v1/reports/oee           # Relatório OEE
POST /api/v1/notifications         # Enviar notificação
```

---

## **9. Futuras Expansões**

### **9.1 Módulo PCP Automático**
```
Entradas:
• Pedidos de venda (ERP)
• Estoque atual (ERP)
• Capacidade das máquinas (sistema)
• Lead times históricos

Saídas:
• Programação automática de produção
• Sugestão de sequenciamento
• Alerta de conflitos/capacidade
• Simulação de cenários
```

### **9.2 Módulo de Manutenção Preditiva**
```
• Coleta de dados dos ESP32
• Análise de vibração/temperatura
• Alerta de desgaste antecipado
• Agenda de manutenção preventiva
• Histórico de falhas
```

### **9.3 Módulo de Qualidade Integrado**
```
• Checkpoints digitais de qualidade
• Amostragem estatística
• Rastreabilidade completa
• Análise de causa-raiz
• Plano de ação 8D
```

### **9.4 Módulo de Competências**
```
• Matriz de habilidades por operador
• Plano de treinamento
• Certificações e validações
• Sugestão de alocação por habilidade
```

---

## **10. Plano de Implementação**

### **Fase 1: MVP (4-6 semanas)**
- [ ] Estrutura básica do sistema
- [ ] Autenticação e perfis
- [ ] Cadastro de máquinas e produtos
- [ ] Apontamento básico (início/fim)
- [ ] Integração com ERP para carregar OPs
- [ ] Relatórios simples

### **Fase 2: Funcionalidades Core (6-8 semanas)**
- [ ] Gestão de paradas com motivos
- [ ] Velocidades específicas por produto
- [ ] Cálculo automático de eficiência
- [ ] Painel Andon básico
- [ ] Notificações em tempo real
- [ ] Dashboard operacional

### **Fase 3: Otimização LEAN (4-6 semanas)**
- [ ] OEE completo
- [ ] Análise de perdas
- [ ] Kanban digital
- [ ] SMED digital
- [ ] Relatórios avançados
- [ ] Treinamento dos usuários

### **Fase 4: Expansão (contínua)**
- [ ] Integração com ESP32
- [ ] PCP automático
- [ ] Manutenção preditiva
- [ ] Módulo de qualidade
- [ ] Aplicativo nativo mobile

---

## **11. Considerações Finais**

### **Fatores Críticos de Sucesso**
1. **Engajamento dos usuários**: Operadores devem ver valor no sistema
2. **Qualidade dos dados**: Velocidades e parâmetros precisos
3. **Integração ERP**: Sincronização confiável e em tempo real
4. **Infraestrutura**: Rede Wi-Fi robusta no chão de fábrica
5. **Suporte e treinamento**: Capacitação contínua

### **Riscos e Mitigações**
| Risco | Probabilidade | Impacto | Mitigação |
|-------|--------------|---------|-----------|
| Resistência à mudança | Alta | Alto | Envolvimento desde o início, demonstração de benefícios |
| Dados inconsistentes no ERP | Média | Alto | Validação cruzada, limpeza inicial |
| Problemas de conectividade | Alta | Médio | Funcionalidade offline, Wi-Fi industrial |
| Velocidades imprecisas | Média | Médio | Ajuste contínuo baseado em dados reais |

### **ROI Esperado**
- **Redução de 40%** no tempo de apontamento
- **Aumento de 5-10%** na eficiência global
- **Redução de 15-20%** no tempo de parada
- **Eliminação de 100%** das planilhas manuais
- **Decisões 50% mais rápidas** com dados em tempo real

---

**Próximos Passos Imediatos:**
1. Validar documentação com stakeholders
2. Priorizar funcionalidades do MVP
3. Definir equipe de desenvolvimento
4. Estabelecer cronograma detalhado
5. Iniciar prototipagem das telas principais

Esta documentação serve como guia completo para o desenvolvimento do **Production Pointer Pro**, garantindo que todas as necessidades da produção têxtil sejam atendidas com foco na metodologia LEAN e na experiência do usuário final.