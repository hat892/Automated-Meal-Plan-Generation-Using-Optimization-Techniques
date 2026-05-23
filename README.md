# Meal Plan Optimizer

An automated daily meal plan generator that uses a **Genetic Algorithm (GA)** to produce nutritionally balanced one-day meal plans aligned with Saudi dietary guidelines. The system simultaneously optimizes for caloric accuracy, macronutrient balance, meal variety, and user food preferences.

---

## How It Works

### Problem Formulation

Meal planning is formulated as a combinatorial optimization problem. The search space consists of all possible selections of food items across 5 meals × 6 food groups = 30 decision variables per solution.

### Solution Representation

Each solution (chromosome) represents one complete daily meal plan:

```
Chromosome = [Breakfast × 6 groups] + [Snack 1 × 6] + [Lunch × 6] + [Snack 2 × 6] + [Dinner × 6]
           = 30 genes
```

Each gene stores a food item name and a serving count. Serving counts are fixed to Saudi dietary guidelines per calorie target — only food item selection evolves.

### Fitness Function

Four penalty terms are combined into a single weighted score:

```
Score   = w1·J_macro + w2·J_var + w3·J_pref + w4·J_cal
Fitness = 1 / (1 + Score)
```

| Penalty | Symbol | Description |
|---|---|---|
| Macronutrient balance | J_macro | Squared error vs. AMDR targets (55% carbs / 20% protein / 25% fats) |
| Meal variety | J_var | Penalizes repeated food items across the day |
| User preference | J_pref | Average dislike score across all food items |
| Caloric accuracy | J_cal | Relative deviation from daily caloric target |

Default weights: w1 = w2 = w3 = w4 = 0.25

### GA Configuration

| Parameter | Value |
|---|---|
| Population size | 95 |
| Crossover | One-point at meal boundaries |
| Crossover probability | 0.55 |
| Mutation | Replace up to 2 food items with meal-compatible alternatives |
| Mutation probability | 0.17 |
| Selection | Tournament (size 3) |
| Strategy | (μ + λ), λ = 100 |
| Early stopping | 50 generations without improvement |
| Max generations | 300 |

---

## Experiments

### Experiment 1 — GA vs. Baseline Algorithms

Evaluates the GA against three baseline algorithms (Random Search, Local Search, Simulated Annealing) under identical time budgets. All baselines run for exactly the same wall-clock time as the GA to ensure a fair comparison.

- **10 calorie targets**: 1,200 – 2,200 kcal/day
- **2 weight configurations**: macro/calorie emphasis vs. equal weighting
- **10 independent runs** per condition
- Statistical significance tested via Wilcoxon signed-rank test (α = 0.05)

### Experiment 2 — Dataset Sensitivity

The original Bahrain food dataset contains only **219 items**, which provides a limited search space and may restrict the ability to differentiate between algorithm performances. Experiment 2 investigates how the **size and composition** of the food dataset affect the GA and its competitors.

Six expanded datasets were constructed using two augmentation strategies:

- **Synthetic generation**: for each food category, the mean and standard deviation of every nutritional attribute are calculated, and new food items are sampled from the resulting normal distribution — producing realistic items without copying existing rows.
- **Duplication with jitter**: existing rows are copied with a small random perturbation of up to 3% applied to each numerical value, preserving the overall distribution while introducing minor variation.

These strategies were applied at two scales and three configurations:

| Case | Size | Augmentation |
|---|---|---|
| Case 1 | 1,000 rows | Original + Synthetic |
| Case 2 | 1,000 rows | Original + Duplicates (with jitter) |
| Case 3 | 5,000 rows | Original + Synthetic |
| Case 4 | 5,000 rows | Original + Duplicates (with jitter) |
| Case 5 | 1,000 rows | Original + Synthetic + Duplicates |
| Case 6 | 5,000 rows | Original + Synthetic + Duplicates |

Each case is evaluated at **1,200 / 1,600 / 2,200 kcal** targets across 10 independent runs per algorithm.

---

## Installation

```bash
git clone https://github.com/your-username/meal-plan-optimizer.git
cd meal-plan-optimizer
pip install deap numpy scipy matplotlib pandas
```

**Requirements:** Python 3.9+

---

## Usage

All notebooks are in the `Code/` folder. All datasets are in the `Datasets/` folder.

### Experiment 1

Open `Code/Meal_Plan_Optimization_GA.ipynb` and run **Kernel → Restart & Run All**.

Expected runtime: **1–3 hours**

### Experiment 2

Open `Code/Experiment_2_Dataset_Sensitivity.ipynb` and run **Kernel → Restart & Run All**.

Expected runtime: **10–20 minutes per case × 6 cases**


---

## Dataset

The base dataset is the **Bahrain Calorie Guideline Dataset** (219 food items), categorized into 6 food groups: Vegetables, Fruits, Grains, Protein, Dairy, and Fats and Oils. Each entry includes:

- Macronutrient breakdown (protein, fat, carbohydrates, calories)
- Meal suitability flags (which meals each food is appropriate for)
- User preference score (0 = extremely liked → 1 = extremely disliked)


