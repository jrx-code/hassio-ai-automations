# 🗑️ AI Waste Bin Verification for Home Assistant

A smart automation that visually verifies if you've remembered to put out the trash bins based on your collection schedule. It uses GenAI (OpenAI GPT-4o/Nano via `ai_task`) to analyze camera snapshots and sends you the result with the snapshot: bins out, bins missing (high priority) or uncertain.

![Vault-Tec Waste-O-Meter](/images/waste-collection.png)


## 🚀 How it works

1.  **Trigger:** Runs automatically at 8:00 AM and 9:00 PM, or manually via a test button (`input_button`).
2.  **Guards:** Stops if maintenance mode (`input_boolean`) is on, or if the next-collection sensor does not say **today** (morning run) / **tomorrow** (evening run).
3.  **Dynamic Context:** Determines the color of the bin (e.g., Blue, Yellow, Black) based on the waste fraction.
4.  **Visual Analysis:**
    * Takes a snapshot from your driveway camera into `/config/www/snapshots/waste_check.jpg` and refreshes a camera helper that shows that file.
    * Sends the helper's image to an `ai_task` entity with instructions to look for the required bin color(s).
5.  **Decision & Notification:** the answer is stored in a counter helper (1 = found, 2 = missing, 3 = uncertain, 5 = unexpected reply) and a notification with the snapshot goes to your phone:
    * ✅ bins found: "Bins Out",
    * 🚨 bins **missing**: high-priority "MISSING BINS",
    * 🤔 uncertain: "Low Confidence", check manually.

## 🛠️ Prerequisites

* **Home Assistant** (Core/OS).
* **AI Integration:** an `ai_task` entity that accepts image attachments (e.g. OpenAI, or [Ollama](https://github.com/ollama/ollama) with a vision model). The automation calls `ai_task.generate_data`.
* **Waste Schedule Sensors:** one sensor per fraction plus one for the next collection, whose state contains `today` or `tomorrow` (e.g., from [Waste Collection Schedule](https://github.com/mampfes/hacs_waste_collection_schedule)).
* **Camera:** A camera pointing at your bin storage/driveway area.
* **Helpers** (create them in *Settings → Devices & services → Helpers*, or rename them in the YAML):
    * `camera.waste_snapshot_proxy` — a camera that shows `/config/www/snapshots/waste_check.jpg` (e.g. the *Local File* integration); the AI reads the image from it,
    * `counter.waste_check_result` — counter holding the last result,
    * `input_button.waste_test_button` — manual test trigger,
    * `input_boolean.try_maintenance` — maintenance switch; while it is **on** the automation does nothing.

## 📥 Installation

1.  Copy the file `automations/waste_collection.yaml` to your Home Assistant `automations` folder (or copy the content into a new automation).
2.  **Adapt the Entity IDs**: You MUST replace the placeholder sensors with your own entity IDs in the YAML file.

### Required Entity Changes:

Search and replace the following in `waste_collection.yaml`:

* `camera.driveway_high_res` -> Your camera entity ID.
* `notify.mobile_app_your_device` -> Your mobile app notification service (used three times).
* `ai_task.gpt_vision_service` -> Your AI task entity ID.
* `sensor.waste_plastic` -> Your sensor for Plastic/Metal collection.
* `sensor.waste_paper` -> Your sensor for Paper collection.
* `sensor.waste_glass` -> Your sensor for Glass collection.
* `sensor.waste_bio` -> Your sensor for Bio/Organic collection.
* `sensor.waste_mixed` -> Your sensor for Mixed/Residual waste.
* `sensor.first_waste_collection` -> A general sensor for the next collection (required: the condition checks it for `today` / `tomorrow`).
* `camera.waste_snapshot_proxy`, `counter.waste_check_result`, `input_button.waste_test_button`, `input_boolean.try_maintenance` -> your helpers (see Prerequisites).

## ⚙️ Configuration Details

### The Logic Logic
The automation sets `target_day` (`tomorrow` for the 21:00 run, otherwise `today`) and builds `waste_data`, a JSON object with `colors` and `fractions`. It checks each fraction sensor: if `sensor.waste_paper` contains the target day, it adds `blue` (paper), and so on (yellow = plastic and metal, blue = paper, green = glass, brown = bio, black = mixed waste). From that it derives `search_colors` (e.g. `blue or yellow`), `search_fractions` and `found_count`; if nothing matches, the run stops.

### The AI Prompt
The prompt sent to the LLM is structured to force a JSON-like strict response:
> "Determine if a waste bin or bag in the following color is present: **{{ search_colors }}**. Respond ONLY with: 'true' (visible), 'false' (missing), or 'uncertain' (unclear)."

## 🤝 Contributing

Feel free to open issues or pull requests if you have ideas to improve the detection prompt or logic!

## 📄 License

Apache License 2.0, see [LICENSE](LICENSE).
