Many-objective NSGA-III diet optimisation — data and code

This repository holds the food-composition dataset and the code for the paper:

T. Shakeel, A. A. Khan, S. Khan. A many-objective NSGA-III framework for mitigating the nutritional risks of calorie restriction.

The method treats the side effects of a calorie deficit (lean-muscle loss, hair thinning, low mood, persistent hunger) as explicit objectives and optimises a full day's intake with NSGA-III over a pool of real USDA foods.

Contents
opt125.py: the full pipeline: data prep, objective and constraint model, NSGA-III search, plan selection, evaluation, and the client plan document
foods_no_zero_energy.csv: merged, cleaned food-composition table (7,964 foods)
README: this file
Requirements

Python 3.10+ and:

pip install numpy pandas scipy matplotlib pymoo psutil

Optional, only for the meal-plan document and recipe text:

pip install python-docx google-genai requests

python-docx writes the client plan as a .docx; google-genai and requests call the Gemini API to draft recipe wording in the rebuild stage. Both are optional; the optimisation and every reported metric run without them.

Data

foods_no_zero_energy.csv: 7,964 foods, 42 columns, values per 100 g. Units are in the headers (KCAL, G, MG = mg, UG = µg).

Identifiers: fdc_id, description
Energy: Energy (Atwater Specific Factors) (KCAL)
Macronutrients: protein, total fat, carbohydrate
Fibre and sugars: total fibre, total sugars, added sugars
Amino acid: leucine
Fatty-acid classes: saturated, trans, monounsaturated; PUFA LA, ALA, EPA, DHA
Micronutrients (23): vitamins A, C, D, E, B6, B12, folate, thiamin, riboflavin, niacin, pantothenic acid, biotin; minerals magnesium, zinc, selenium, iron, potassium, calcium, copper, iodine, manganese, molybdenum, phosphorus; plus sodium

Built from USDA FoodData Central (public domain) by merging Foundation Foods and SR Legacy, harmonising energy to kcal (Atwater specific factors; kilojoule entries converted at 4.184 kJ/kcal), dropping rows with no energy value, and setting missing nutrient cells to zero.

Data quality notes
A 0 can mean "reported zero" or "not measured," because missing values were set to zero.
Sparse columns (under 40% populated): LA, ALA, EPA, DHA, trans fat, added sugars, vitamin D, biotin, iodine, molybdenum. Biotin, iodine and molybdenum are non-zero in under 1% of foods, so any score leaning on them is unreliable.
Sugars, added and Added Sugars (g) are identical in every row, so one is a duplicate; keep one.
This is the table before the pipeline's own filters, which reduce it to the 2,307-food candidate pool used in the paper.
Running it

The script is interactive:

python opt125.py

It prompts for: the user profile (sex, age, weight, height, activity level, GLP-1 or weight-loss flag); the food CSV path (enter foods_no_zero_energy.csv; multiple comma-separated paths are accepted); the serving grid in grams (default 5); and the run type, where you press Enter for a research-grade run or choose the quick trial (n_gen 80, 2 seeds). Optional prompts follow for a score-equation sensitivity analysis, a crossover and mutation operator sweep, reusing checkpoints, and a Gemini API key if you want recipe text.

Settings used in the paper
Algorithm: NSGA-III with Riesz s-energy reference directions; population 210
Generations: 3,000 by default; the primary profile in the paper used 12,000
Seeds: 21 (primary profile)
Crossover: SBX, distribution index 30, probability 1.0
Mutation: polynomial, distribution index 40, probability 1/20 (1 / maximum active foods)
Serving grid: 5 g; cardinality: 5 to 20 active foods
Random baseline: 2,000 feasible diets
Selection: equal-weight multi-criteria rule over the pooled front
Outputs (written to the run folder)
pareto_front.csv: pooled non-dominated plans
recommended_diet_plan.csv: the selected plan
recommended_meal_split.csv: the plan split into five meals
recommended_plan_watchlist.csv: nutrients sitting near their limits
pareto_front_pcp.png: parallel-coordinates view of the front
convergence_hv.csv, convergence_hv.png, convergence_rate_hv.png: hypervolume traces
nutrient_sensitivity.csv and .png: if the sensitivity analysis is run
diet_plan_client.docx or rebuilt_diet_plan.docx, and adjustments_log.csv: the issued plan (needs python-docx)
License

Choose and state a license. Code: MIT is common for research code. Derived data: USDA FoodData Central is public domain.

Contact

Tazayun Shakeel (corresponding author), tazayunbhat2020@cukashmir.edu.in
Department of Information Technology, School of Engineering & Technology, Central University of Kashmir, Ganderbal 191131, Jammu and Kashmir, India
