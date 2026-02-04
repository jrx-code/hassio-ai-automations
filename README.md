# 🗑️ AI Waste Bin Verification for Home Assistant

A smart automation that visually verifies if you've remembered to put out the trash bins based on your collection schedule. It uses GenAI (OpenAI GPT-4o/Nano via `ai_task`) to analyze camera snapshots and notifies you only if the bins are missing.

![Vault-Tec Waste-O-Meter](/images/waste_pipboy.jpg)


## 🚀 How it works

1.  **Trigger:** Runs automatically at 8:00 AM and 9:00 PM (or manually).
2.  **Schedule Check:** Checks if there is a waste collection scheduled for **Today** or **Tomorrow**.
3.  **Dynamic Context:** Determines the color of the bin (e.g., Blue, Yellow, Black) based on the waste fraction.
4.  **Visual Analysis:**
    * Takes a snapshot from your driveway camera.
    * Sends the image to an AI Agent with specific instructions to look for the required bin color.
5.  **Decision & Notification:**
    * If the bin is found: ✅ Logs success.
    * If the bin is **missing**: 🚨 Sends a high-priority actionable notification to your phone with the snapshot.

## 🛠️ Prerequisites

* **Home Assistant** (Core/OS).
* **AI Integration:** Custom integration capable of image analysis (e.g., [Ollama](https://github.com/ollama/ollama) or OpenAI via `ai_task` or standard HA conversation agents).
* **Waste Schedule Sensors:** Sensors that return `today` or `tomorrow` as state (e.g., from [Waste Collection Schedule](https://github.com/mampfes/hacs_waste_collection_schedule)).
* **Camera:** A camera pointing at your bin storage/driveway area.

## 📥 Installation

1.  Copy the file `automation/waste_collection.yaml` to your Home Assistant `automations` folder (or copy the content into a new automation).
2.  **Adapt the Entity IDs**: You MUST replace the placeholder sensors with your own entity IDs in the YAML file.

### Required Entity Changes:

Search and replace the following in `waste_collection.yaml`:

* `camera.driveway_camera` -> Your camera entity ID.
* `notify.mobile_app_your_phone` -> Your mobile app notification service.
* `ai_task.openai_gpt_5_nano` -> Your AI model entity ID.
* `sensor.waste_plastic` -> Your sensor for Plastic/Metal collection.
* `sensor.waste_paper` -> Your sensor for Paper collection.
* `sensor.waste_glass` -> Your sensor for Glass collection.
* `sensor.waste_bio` -> Your sensor for Bio/Organic collection.
* `sensor.waste_mixed` -> Your sensor for Mixed/Residual waste.
* `sensor.next_waste_collection` -> A general sensor showing the next collection date (optional, used in condition).

## ⚙️ Configuration Details

### The Logic Logic
The automation creates a variable `target_colors` (translated from `szukane_kolory`). It iterates through your sensors. If `sensor.waste_paper` is 'tomorrow', it appends `blue` to the list of things to look for.

### The AI Prompt
The prompt sent to the LLM is structured to force a JSON-like strict response:
> "Look for a container/bag in color: **[target_colors]**. Reply strictly: 'true', 'false', or 'uncertain'."

## 🤝 Contributing

Feel free to open issues or pull requests if you have ideas to improve the detection prompt or logic!

## 📄 License

MIT
