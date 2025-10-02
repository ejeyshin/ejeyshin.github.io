# SageMaker Hyperparameter Tuning Cost Optimization

**Exam Domain**: Domain 3 - Modeling (36%) - Task Statement 3.4: Perform hyperparameter optimization

## 🎯 Key Concept

Reducing hyperparameter tuning costs in SageMaker requires understanding the relationship between job parallelism, search strategies, and parameter space exploration.

## ✅ Cost Reduction Strategies

### 1. **Decrease Concurrent Hyperparameter Tuning Jobs**

- **Why it works**: Fewer concurrent jobs = fewer EC2 instances running simultaneously
- **Configuration**: Set `max_parallel_jobs` to a lower value
- **Trade-off**: Longer total time, but significantly lower cost
- **When to use**: When time is not critical, budget is constrained

```python
# Example configuration
hyperparameter_tuner = HyperparameterTuner(
    estimator=estimator,
    objective_metric_name='validation:accuracy',
    hyperparameter_ranges=hyperparameter_ranges,
    max_jobs=100,
    max_parallel_jobs=5,  # ← Reduce this for cost savings
    strategy='Bayesian'
)
```

### 2. **Use Logarithmic Scales on Parameter Ranges**

- **Why it works**: Efficiently explores parameters that span multiple orders of magnitude
- **Result**: Fewer trials needed to find optimal values = lower cost
- **Best for**: Learning rates, regularization parameters, batch sizes

```python
# Logarithmic scale example
hyperparameter_ranges = {
    'learning_rate': ContinuousParameter(1e-4, 1e-1, scaling_type='Logarithmic'),
    'l2_regularization': ContinuousParameter(1e-5, 1e-2, scaling_type='Logarithmic')
}
```

**Comparison**:
- Linear scale: 0.0001, 0.00028, 0.00046, 0.00064... (inefficient)
- Log scale: 0.0001, 0.001, 0.01, 0.1 (efficient exploration)

## ❌ What NOT to Do

### 1. **Don't Disable Warm Start**
- Warm start REDUCES cost by reusing knowledge from previous tuning jobs
- Disabling it means starting from scratch = more trials = higher cost

### 2. **Don't Use Grid Search**
- Grid search explores EVERY combination (exponentially expensive)
- **Example**: 3 params × 10 values = 10³ = 1,000 trials
- Use Bayesian or Random search instead

### 3. **Don't Disable Logarithmic Scales When Appropriate**
- Reverse log scales are rarely used; disabling them has no impact
- Keep log scales for parameters that vary by orders of magnitude

## 🔑 Key Takeaways

1. **Parallelism vs Cost**: More concurrent jobs = faster results but higher cost
2. **Smart Search**: Logarithmic scales + Bayesian strategy = fewer trials
3. **Warm Start**: Always enable when you have previous tuning jobs
4. **Avoid Grid Search**: Use for small parameter spaces only (<100 combinations)

## 📊 Cost Formula Understanding

```
Total Cost = (# of training jobs) × (instance cost per hour) × (training time)

Reduce cost by:
- Decreasing concurrent jobs (spreads cost over time)
- Using log scales (reduces total jobs needed)
- Enabling warm start (reduces total jobs needed)
```

## 🔗 Related Concepts

- SageMaker Automatic Model Tuning
- Bayesian Optimization
- Hyperparameter search strategies
- Resource optimization in ML training

## 📚 References

- [SageMaker Hyperparameter Tuning Best Practices](https://docs.aws.amazon.com/sagemaker/latest/dg/automatic-model-tuning.html)
- AWS ML Specialty Exam Guide - Domain 3: Modeling

---

## 📝 Advanced Practice Questions

### Question 1: Budget-Constrained Tuning

You're running a SageMaker hyperparameter tuning job with 5 hyperparameters. Your budget is limited, but you need results within 24 hours. Which strategy provides the best balance?

A) Grid search with max_parallel_jobs=50  
B) Random search with max_parallel_jobs=10 and max_jobs=200  
C) Bayesian optimization with max_parallel_jobs=5 and max_jobs=100  
D) Grid search with max_parallel_jobs=5  

<details>
<summary>Answer</summary>

**C) Bayesian optimization with max_parallel_jobs=5 and max_jobs=100**

**Why:**

**Grid search is impossible:**
- 5 hyperparameters with just 5 values each = 5^5 = 3,125 combinations
- With 10 values each = 10^5 = 100,000 combinations
- Way beyond budget and time constraints

**Cost comparison (assume $2/hour instances, 30-min jobs):**
- A) 50 parallel impossible to complete (too many combinations)
- B) Random: 200 jobs × 0.5 hours × $2 = $200
- C) Bayesian: 100 jobs × 0.5 hours × $2 = $100 ✅ (best cost)
- D) Grid: Can't complete in time

**Time comparison:**
- B) 200 ÷ 10 = 20 batches × 30 min = 10 hours ✅
- C) 100 ÷ 5 = 20 batches × 30 min = 10 hours ✅

**Why Bayesian wins:**
- Uses past results to guide next trials (smarter search)
- Needs fewer total trials than Random for same quality
- Lower parallelism = lower cost, still meets 24-hour deadline
- Typically finds optimal in 50-150 trials for 5 hyperparameters

**Key insight:** High-dimensional spaces (5+ params) + budget constraints = Bayesian, not Grid
</details>

---

### Question 2: Real-Time Model Retraining

Your fraud detection model retrains every 4 hours using hyperparameter tuning. Each training job takes 15 minutes. You need tuning to complete within 2 hours to deploy updated model. Current setup uses Bayesian with max_parallel_jobs=8, max_jobs=50, costing $48 per tuning cycle.

New requirement: Reduce cost by 50% while maintaining 2-hour deadline.

Which approach works?

A) Reduce max_parallel_jobs to 4, keep max_jobs=50  
B) Enable warm start from previous cycle, reduce max_jobs to 25  
C) Switch to Random search, max_parallel_jobs=4, max_jobs=30  
D) Use Spot instances with max_parallel_jobs=8, max_jobs=50  

<details>
<summary>Answer</summary>

**B) Enable warm start from previous cycle, reduce max_jobs to 25**

**Why:**

**Current state:**
- 50 jobs ÷ 8 parallel = 6.25 batches × 15 min = 94 minutes
- Cost: 50 jobs × 0.25 hours × instance cost = $48

**Option analysis:**

**A) Reduce parallel to 4:**
- 50 ÷ 4 = 12.5 batches × 15 min = 188 min (3+ hours) ❌ Misses deadline

**B) Warm start + reduce jobs:**
- Warm start reuses knowledge from 4 hours ago (model drift minimal)
- Needs ~50% fewer trials: 25 jobs
- 25 ÷ 8 parallel = 3.1 batches × 15 min = 47 minutes ✅
- Cost: 25 × 0.25 = 6.25 instance-hours (50% of original) ✅

**C) Random search with fewer jobs:**
- Random less efficient than Bayesian (needs more trials)
- 30 ÷ 4 = 7.5 batches × 15 min = 112 minutes ✅ (meets time)
- But Random may miss optimal compared to Bayesian with warm start

**D) Spot instances:**
- 70% savings: $48 × 0.3 = $14.40 ✅ (exceeds 50% reduction)
- BUT: 15-min jobs very risky for interruptions
- Fraud detection = critical, can't afford failed tuning

**Key pattern:** Frequent retraining (every 4 hours) = Perfect use case for warm start
</details>

---

### Question 3: Multi-Region Deployment

You're deploying a recommendation model across 3 regions. Each region needs hyperparameter tuning for local data characteristics. Each tuning job: 120 trials, 20 minutes per trial, $3/hour instances.

Current approach: Run 3 separate tuning jobs sequentially.
- Region 1: 120 trials
- Region 2: 120 trials  
- Region 3: 120 trials

Which optimization reduces cost the most?

A) Run all 3 regions in parallel with max_parallel_jobs=10 each  
B) Use transfer learning: Full tuning in Region 1, then warm start for Regions 2 & 3 with 40 trials each  
C) Use same hyperparameters across all regions (no region-specific tuning)  
D) Reduce to 60 trials per region with higher max_parallel_jobs=20  

<details>
<summary>Answer</summary>

**B) Transfer learning approach with warm start**

**Why:**

**Current cost:**
- 3 regions × 120 trials × 0.33 hours × $3 = $360 total

**Option analysis:**

**A) Parallel execution:**
- Same 360 trials total, same cost $360 ❌
- Just faster completion, not cheaper

**B) Transfer learning with warm start:**
- Region 1 (source): 120 trials × 0.33 × $3 = $120
- Region 2 (warm start): 40 trials × 0.33 × $3 = $40
- Region 3 (warm start): 40 trials × 0.33 × $3 = $40
- **Total: $200** ✅ (44% cost reduction)

**C) No region-specific tuning:**
- Zero cost for tuning ✅
- BUT: Poor model performance in Regions 2 & 3
- Recommendation systems need local optimization (language, preferences differ)
- Lost revenue > saved cost ❌

**D) Reduce trials everywhere:**
- 3 × 60 × 0.33 × $3 = $180
- Saves money but may miss optimal
- No intelligence about regional similarities

**Why B is best:**
- Region 1 finds globally good hyperparameters
- Regions 2 & 3 only need local fine-tuning (fewer trials)
- Assumes regions have similar characteristics (usually true)
- 40 trials enough for local adaptation

**Key insight:** Multi-region/multi-dataset scenarios = Use warm start with transfer learning pattern
</details>

---

### Question 4: Parameter Space Explosion

Training a computer vision model with these hyperparameters:
- Learning rate: 0.0001 to 0.1 (continuous)
- Batch size: 16, 32, 64, 128, 256 (5 values)
- Optimizer: Adam, SGD, RMSprop (3 values)
- Weight decay: 0.00001 to 0.01 (continuous)
- Dropout: 0.1 to 0.5 (continuous)

Each job takes 45 minutes on ml.p3.2xlarge ($3.06/hour). You have $500 budget and 48-hour deadline.

Which configuration maximizes model quality within constraints?

A) Grid search on discrete params only, linear scale on continuous params, max_parallel_jobs=20  
B) Random search with max_jobs=300, max_parallel_jobs=15  
C) Bayesian with log scale on learning_rate and weight_decay, max_jobs=150, max_parallel_jobs=10  
D) Two-phase: Random search (50 trials) then Bayesian warm start (100 trials), max_parallel_jobs=8  

<details>
<summary>Answer</summary>

**D) Two-phase approach: Random then Bayesian with warm start**

**Why:**

**Budget calculation:**
- $500 ÷ $3.06/hour ÷ 0.75 hours per job = ~217 jobs maximum

**Time calculation (must fit in 48 hours = 2,880 minutes):**

**Option analysis:**

**A) Grid search on discrete:**
- Just batch_size and optimizer = 5 × 3 = 15 combinations
- Still have 3 continuous parameters (infinite combinations)
- Can't do full grid, unclear strategy ❌

**B) Random search 300 jobs:**
- Cost: 300 × 0.75 × $3.06 = $688 ❌ (over budget)

**C) Bayesian 150 jobs:**
- Cost: 150 × 0.75 × $3.06 = $344 ✅ (under budget)
- Time: 150 ÷ 10 = 15 batches × 45 min = 675 min = 11.25 hours ✅
- Log scaling good for learning_rate and weight_decay ✅

**D) Two-phase (50 Random + 100 Bayesian):**
- Cost: 150 × 0.75 × $3.06 = $344 ✅
- Time: 150 ÷ 8 = 18.75 batches × 45 min = 844 min = 14 hours ✅
- **Phase 1 (Random 50)**: Explores broad space quickly
- **Phase 2 (Bayesian 100)**: Exploits promising regions from Phase 1
- Warm start connects phases seamlessly

**Why two-phase is best:**
- Random search: Good for initial exploration (finds promising regions)
- Bayesian: Good for exploitation (refines best regions)
- Combined: Better than either alone for complex spaces
- 5 hyperparameters = medium-high dimensional (benefits from hybrid)

**Implementation:**
```python
# Phase 1: Random exploration
random_tuner = HyperparameterTuner(
    strategy='Random',
    max_jobs=50,
    max_parallel_jobs=8
)

# Phase 2: Bayesian exploitation
bayesian_tuner = HyperparameterTuner(
    strategy='Bayesian',
    max_jobs=100,
    max_parallel_jobs=8,
    warm_start_config=WarmStartConfig(
        warm_start_type='TransferLearning',
        parents=[random_tuner.latest_tuning_job.name]
    )
)
```

**Key insight:** Complex parameter spaces (5+ params, mix of discrete/continuous) = Two-phase strategy beats single-strategy
</details>

---

### Question 5: Spot Instance Risk Assessment

You're tuning a natural language model. Training jobs take 2 hours each on ml.p3.8xlarge instances. You have 72-hour deadline and want to minimize cost.

Scenario details:
- Bayesian optimization: 60 trials needed
- On-demand cost: $12.24/hour
- Spot cost: $3.67/hour (70% savings)
- Historical spot interruption rate for ml.p3.8xlarge: 15% per hour

Which strategy balances cost and reliability?

A) All Spot instances with max_parallel_jobs=10, max_wait_time=6 hours  
B) All On-Demand instances with max_parallel_jobs=5  
C) Hybrid: First 30 trials On-Demand, last 30 trials Spot with max_wait_time=12 hours  
D) All Spot with max_parallel_jobs=3, max_wait_time=24 hours  

<details>
<summary>Answer</summary>

**A) All Spot instances with max_parallel_jobs=10, max_wait_time=6 hours**

**Why:**

**Interruption risk calculation:**
- 2-hour job with 15% interruption/hour
- Probability of completing without interruption: (0.85)^2 = 72%
- Probability of interruption: 28% per job
- With 60 trials: Expected interruptions = 60 × 0.28 = ~17 interruptions

**Deadline buffer calculation:**
- Optimal time: 60 ÷ 10 = 6 batches × 2 hours = 12 hours
- Deadline: 72 hours
- Buffer ratio: 72 ÷ 12 = 6x ✅ (very safe, >3x recommended)

**Option analysis:**

**A) 10 parallel Spot, 6-hour max_wait:**
- Optimal: 12 hours
- With ~17 retries: Add ~3-4 hours (retries spread across parallel jobs)
- Total: ~15-16 hours ✅ (well under 72-hour deadline)
- Cost: ~70 job executions × 2 × $3.67 = $514 ✅
- 6-hour max_wait allows 2-3 retry attempts per job ✅

**B) 5 parallel On-Demand:**
- Time: 60 ÷ 5 = 12 batches × 2 hours = 24 hours ✅
- Cost: 60 × 2 × $12.24 = $1,469 ❌ (3x more expensive)
- Reliable but unnecessarily expensive given generous deadline

**C) Hybrid approach:**
- First 30 On-Demand: 30 × 2 × $12.24 = $734
- Last 30 Spot: 30 × 2 × $3.67 = $220
- Total: $954 (saves vs B, but costs more than A)
- No technical benefit over all-Spot with this deadline

**D) 3 parallel Spot, 24-hour max_wait:**
- Optimal: 60 ÷ 3 = 20 batches × 2 hours = 40 hours
- With retries: ~52-55 hours (still under 72) ✅
- Cost: ~$514 (same as A)
- Unnecessarily slow when deadline allows higher parallelism

**Why A is best:**
- Deadline buffer (6x) makes Spot very safe
- High parallelism (10) completes fast with buffer for retries
- 6-hour max_wait appropriate for 2-hour jobs
- Saves ~$950 compared to On-Demand

**Spot viability formula:**
```
Safe to use Spot when: deadline ÷ optimal_time ≥ 3

This scenario: 72 ÷ 12 = 6 ✅✅ (very safe)
```

**Key insight:** Generous deadline (>3x optimal time) + moderate interruption rate (<20%/hour) = Spot is optimal choice
</details>

---

### Question 6: Incremental Hyperparameter Addition

You have an existing production model with 3 tuned hyperparameters achieving 92% accuracy. Data science team wants to add 2 new hyperparameters to potentially improve to 95%.

Existing hyperparameters (already optimized):
- Learning rate: 0.001 (from range 0.0001-0.01)
- L2 regularization: 0.0001 (from range 0.00001-0.001)
- Batch size: 64 (from range 16-256)

New hyperparameters to explore:
- Label smoothing: 0.0 to 0.2
- Mixup alpha: 0.0 to 1.0

Original tuning job: 200 trials, $800 cost. New budget: $400.

Best approach?

A) Full retuning: All 5 hyperparameters from scratch, 200 trials  
B) Warm start: Use previous job as parent, tune all 5 hyperparameters, 100 trials  
C) Fixed existing: Keep 3 optimized values, only tune 2 new hyperparameters, 50 trials  
D) Two-stage: Tune 2 new params alone (30 trials), then fine-tune all 5 together (50 trials)  

<details>
<summary>Answer</summary>

**B) Warm start from previous job, tune all 5 hyperparameters, 100 trials**

**Why:**

**The trap in option C (Fixed existing):**
- Seems logical: existing params already optimal
- **Fatal flaw**: Hyperparameter interactions!
- Label smoothing + mixup may change optimal learning rate
- New regularization (label smoothing) may change optimal L2
- You'll get suboptimal results

**Cost analysis:**
- A) 200 trials = $800 ❌ (over budget)
- B) 100 trials = $400 ✅ (meets budget exactly)
- C) 50 trials = $200 (under budget but risky)
- D) 80 trials = $320 (under budget)

**Why B beats D (two-stage):**

**Option D problems:**
- Stage 1: Tune label_smoothing + mixup_alpha alone
  - But with what values for learning_rate, L2, batch_size?
  - If using old optimal (0.001, 0.0001, 64), might not be optimal for new params
- Stage 2: Fine-tune all 5
  - Only 50 trials for 5-dimensional space = sparse coverage

**Option B advantages:**
- Warm start "remembers" that learning_rate≈0.001 was good
- But allows adjustment if label_smoothing changes the landscape
- 100 trials for 5 params with warm start > 50 trials for 2 params without context
- Properly explores hyperparameter interactions

**Warm start mechanism:**
- Previous 200 trials inform Bayesian prior
- Search focuses on regions near previous optimum
- But explores how new params affect overall landscape
- Typically needs 40-60% fewer trials than from scratch

**Real example:**
```python
new_hyperparameter_ranges = {
    'learning_rate': ContinuousParameter(1e-4, 1e-2, scaling_type='Logarithmic'),
    'l2_regularization': ContinuousParameter(1e-5, 1e-3, scaling_type='Logarithmic'),
    'batch_size': CategoricalParameter([16, 32, 64, 128, 256]),
    'label_smoothing': ContinuousParameter(0.0, 0.2),  # NEW
    'mixup_alpha': ContinuousParameter(0.0, 1.0)       # NEW
}

tuner = HyperparameterTuner(
    estimator=estimator,
    hyperparameter_ranges=new_hyperparameter_ranges,
    max_jobs=100,
    warm_start_config=WarmStartConfig(
        warm_start_type=WarmStartTypes.IDENTICAL_DATA_AND_ALGORITHM,
        parents={'original-tuning-job-name'}
    )
)
```

**Key insight:** Adding hyperparameters to existing model = Always use warm start, always re-tune all parameters (not just new ones) due to interactions
</details>

---

## 💡 Study Tips & Patterns

### Cost Optimization Decision Tree

```
Need to reduce HPO costs?
│
├─ Time flexible (>48 hours)?
│  └─ Reduce max_parallel_jobs (spreads cost over time)
│
├─ Have previous tuning job?
│  └─ Enable warm start (reduces total trials needed)
│
├─ Parameters span orders of magnitude?
│  └─ Use logarithmic scaling (more efficient exploration)
│
├─ Many hyperparameters (5+)?
│  └─ Use Bayesian, not Grid (exponential cost reduction)
│
└─ Generous deadline (>3x optimal time)?
   └─ Consider Spot instances (70% cost savings)
```

### Key Formulas to Remember

**1. Total Cost:**
```
Cost = num_jobs × job_duration_hours × instance_cost_per_hour
```

**2. Time to Complete:**
```
Time = (num_jobs ÷ max_parallel_jobs) × job_duration
```

**3. Spot Viability:**
```
Safe when: deadline ÷ optimal_time ≥ 3
```

**4. Grid Search Explosion:**
```
Total combinations = param1_values × param2_values × ... × paramN_values
```

### Common Exam Patterns

1. **High-dimensional space + budget constraint** = Bayesian, never Grid
2. **Frequent retraining (daily/hourly)** = Always use warm start
3. **Multi-region/dataset deployment** = Transfer learning with warm start
4. **Tight deadline + critical workload** = On-Demand, not Spot
5. **Adding new hyperparameters** = Warm start + tune ALL params (not just new)
6. **Parameters spanning 1000x+ range** = Use logarithmic scaling

### Quick Reference Table

| Scenario | Best Strategy | Key Reason |
|----------|---------------|------------|
| 5+ hyperparameters | Bayesian | Grid = exponential cost |
| Previous tuning exists | Warm start | 40-60% fewer trials |
| Learning rate 0.0001-0.1 | Log scale | Spans 1000x range |
| Deadline >3x optimal | Spot instances | 70% savings, safe buffer |
| Every 4 hours retraining | Warm start | Minimal drift, reuse knowledge |
| Small space (<100 combos) + compliance | Grid search | Only case where Grid makes sense |
| Complex space (5+ params) | Two-phase: Random→Bayesian | Best exploration + exploitation |

---

## 🎯 Exam Success Checklist

- [ ] Understand cost formula: jobs × duration × instance_cost
- [ ] Know when Grid search is appropriate (almost never!)
- [ ] Recognize warm start opportunities (previous jobs, frequent retraining)
- [ ] Identify parameters needing log scale (>100x range)
- [ ] Calculate deadline buffer for Spot viability (÷3 rule)
- [ ] Spot hyperparameter interactions (don't fix old params when adding new)
- [ ] Remember parallelism trade-off: speed vs cost
- [ ] Know two-phase strategy: Random exploration → Bayesian exploitation

