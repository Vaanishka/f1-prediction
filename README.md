┌─────────────────────────────────────────────────────────────┐
│  OFFLINE: Feature Extraction (Run once per race)           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  python extract_race_features.py --race "Abu Dhabi" --year 2025
│                                                              │
│  1. Load FP2 + Q from FastF1 cache                         │
│  2. For each driver:                                        │
│     ├─ Load telemetry (filtered channels only)            │
│     ├─ Extract 50+ features                                │
│     ├─ Add constructor standings                           │
│     ├─ Add data_quality_flag                               │
│     ├─ del telemetry, del lap_data                        │
│     └─ gc.collect() every 5 drivers                        │
│  3. Save: data/features/2025_abu_dhabi_features.parquet    │
│  4. del session, gc.collect()                              │
│                                                              │
│  Output: 20 rows × 55 columns, <1MB file                   │
│  RAM Peak: 200-400MB (vs 1-2GB before)                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  OFFLINE: Generate Predictions (Run before race weekend)    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  python predict_from_features.py --race "Abu Dhabi" --year 2025
│                                                              │
│  1. Load data/features/2025_abu_dhabi_features.parquet     │
│  2. Load trained model                                      │
│  3. Make predictions (<1 second)                            │
│  4. Save to cache/predictions/2025_abu_dhabi.json          │
│                                                              │
│  Output: JSON with predictions + confidence intervals       │
│  RAM: <100MB                                                │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  ONLINE: Website API (Instant response)                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  GET /api/predict/abu-dhabi/2025                           │
│                                                              │
│  1. Check cache: cache/predictions/2025_abu_dhabi.json     │
│  2. If exists: return immediately (<10ms)                   │
│  3. If missing: return 404 "Predictions not ready"         │
│                                                              │
│  NO FastF1, NO telemetry, NO model inference               │
│  RAM: <50MB                                                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  POST-RACE: Persist to Database                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  python persist_race_results.py --race "Abu Dhabi" --year 2025
│                                                              │
│  1. Load predictions from cache                             │
│  2. Load actual results from FastF1                        │
│  3. Calculate accuracy metrics                              │
│  4. Insert into PostgreSQL/SQLite:                         │
│     ├─ races table                                          │
│     ├─ predictions table                                    │
│     ├─ actual_results table                                │
│     └─ accuracy_metrics table                              │
│                                                              │
│  Future queries read ONLY from database                     │
└─────────────────────────────────────────────────────────────┘
