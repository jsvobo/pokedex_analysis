# The Complete Pokédex Dataset
Simple web scraping example for collecting Pokémon stats from a Pokédex site. This dataset is available at Kaggle under [The Complete Pokédex Dataset](https://www.kaggle.com/cristobalmitchell/pokedex) 

## Imagesets
The output imagesets can be found in the images folder and includes the following .png files
* **small_images:** This image set contain all 898 primary pokemon forms with 215 x 215 pixel dimensions
* **large_images:** This image set contain all 898 primary pokemon forms ranging from 300 x 300 to 1280 x 1280 pixel dimensions
* **alt_images:** This image set contain all pokemon variant forms ranging from 215 x 215 to 1280 x 1280 pixel dimensions

## Dataset
The output dataset file pokemon.csv contains all 898 pokémon from Gen I - Gen VIII and contains the following fields

* **national_number:** The entry number of the Pokémon in the National Pokédex
* **gen:** The numbered generation which the Pokémon was first introduced
* **english_name:** The English name of the Pokémon
* **japanese_name:** The Original Japanese name of the Pokémon
* **primary_type:** The Primary Type of the Pokémon
* **secondary_type:** The Secondary Type of the Pokémon
* **classification:** The Classification of the Pokémon as described by the Sun and Moon or Sword and Shield Pokédex
* **percent_male:** The percentage of the species that are male (Blank if the Pokémon is genderless)
* **percent_female:** The percentage of the species that are female (Blank if the Pokémon is genderless)
* **height_m:** Height of the Pokémon in metres
* **weight_kg:** The Weight of the Pokémon in kilograms
* **capture_rate:** Capture Rate of the Pokémon
* **baseeggsteps:** The number of steps required to hatch an egg of the Pokémon
* **hp:** The Base HP of the Pokémon
* **attack:** The Base Attack of the Pokémon
* **defense:** The Base Defense of the Pokémon
* **sp_attack:** The Base Special Attack of the Pokémon
* **sp_defense:** The Base Special Defense of the Pokémon
* **speed:** The Base Speed of the Pokémon
* **abilities_*:** Four features that denote abilities that the Pokémon is capable of having
* **against_*:** Eighteen features that denote the amount of damage taken against an attack of a particular type
* **is_sublegendary:** Denotes if the Pokémon is sublegendary
* **is_legendary:** Denotes if the Pokémon is legendary
* **is_mythical:** Denotes if the Pokémon is mythical
* **num_abilities:** Number of abilities from 0-4
* **evochain_*:** Seven features that indicate the evolutionary chain and triggers
* **gigantamax:** Form of Pokémon if gigantamax capable
* **mega_evolution:** Form of Pokémon if mega evolution capable
* **mega_evolution_alt:** Alternative form of Pokémon if mega evolution capable
* **description:** Pokédex description from official Pokémon website

## Filtered data types
* **national_number:** Index number
* **gen:** The numbered generation which the Pokémon was first introduced
* **primary_type:** The Primary Type of the Pokémon (18 types)
* **secondary_type:** The Secondary Type of the Pokémon (19 types)
* **percent_male:** The percentage of the species that are male (0 if the Pokémon is genderless)
* **percent_female:** The percentage of the species that are female (0 if the Pokémon is genderless)
* **height_m:** Height of the Pokémon in metres
* **weight_kg:** The Weight of the Pokémon in kilograms
* **capture_rate:** Capture Rate of the Pokémon __(not %! absolute value of some inner point system)__
* **baseeggsteps:** The number of steps required to hatch an egg of the Pokémon
* **hp:** The Base HP of the Pokémon
* **attack:** The Base Attack of the Pokémon
* **defense:** The Base Defense of the Pokémon
* **sp_attack:** The Base Special Attack of the Pokémon
* **sp_defense:** The Base Special Defense of the Pokémon
* **speed:** The Base Speed of the Pokémon
* **against_*:** Eighteen features that denote the amount of damage taken against an attack of a particular type
* **evo_length:** Length of evolution chain (with level triggers) 0-6
* **has_mega_evolution:** 1 if has **mega_evolution** or **mega_evolution_alt** in the original dataset, else 0
* **has_gigantamax:** 1 if has **gigantamax** in the original dataset, else 0
* **rarity:** Column 0-3 either normal, sublegendary, legendary or mythical in the original dataset.

#### Dropped columns:
-  **english_name**
-  **japanese_name**
-  **classification** (was 640 classes)
-  **abilities_*:**
-  **is_*:** (legendary etc.)
-  **evochain_*:**
-  **gigantamax**
-  **mega_evolution:** (both possible, normal+ alt)

