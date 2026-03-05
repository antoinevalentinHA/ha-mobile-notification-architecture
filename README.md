# Home Assistant — Mobile Device Abstraction Layer (Notifications + Trackers + Wi-Fi)

A small, fully native Home Assistant layer that **decouples your automations from specific phones**.

It provides:
- a **stable notification API** (scripts)
- a **single source of truth** for phone identity (helpers)
- automatic **derivation of notify / tracker / Wi-Fi entities**
- optional **stable “dynamic” template sensors** to trigger on Wi-Fi BSSID even when phone entity_ids change

---

## Naming (public / generic)

This repository uses generic user keys:

- `user_1`
- `user_2`

Optional display helpers (human-friendly names):

- `input_text.user_1_name`
- `input_text.user_2_name`

Phone slug helpers (single source of truth):

- `input_text.user_1_phone`
- `input_text.user_2_phone`

Derived helpers (auto-filled by the automation):

- `input_text.user_1_notify`
- `input_text.user_1_tracker`
- `input_text.user_1_wifi_bssid`
- (same for `user_2`)

---

## Why

Many Home Assistant setups hardcode mobile services inside automations:

```yaml
service: notify.mobile_app_pixel_7a
````

Problems:

* changing phone ⇒ refactor many automations
* device names leak everywhere
* duplicated notification logic
* poor portability

---

## Architecture

### Notification path

```
Automations / Logic
      │
      ▼
script.notification_*
      │
      ▼
notify.mobile_app_<phone>
      │
      ▼
Phone
```

### Phone identity (single source of truth)

```
input_text.user_<n>_phone      # "pixel_7a"
      │
      ▼
Automation derives:
  input_text.user_<n>_notify      -> "notify.mobile_app_pixel_7a"
  input_text.user_<n>_tracker     -> "device_tracker.pixel_7a"
  input_text.user_<n>_wifi_bssid  -> "sensor.pixel_7a_wi_fi_bssid"
```

### Optional stable Wi-Fi “dynamic” sensor

```
input_text.user_<n>_wifi_bssid        -> "sensor.pixel_7a_wi_fi_bssid"
      │
      ▼
sensor.user_<n>_bssid_dynamic         -> states("sensor.pixel_7a_wi_fi_bssid")
```

---

## Repository contents

```
helpers/
  phones_helpers.yaml

automations/
  phones_sync_derived_helpers.yaml

scripts/
  mobile_notifications.yaml

template_sensors/
  phones_bssid_dynamic.yaml
```

---

## Example usage

Automation calls a script (never `notify.mobile_app_*` directly):

```yaml
service: script.notification_send
data:
  target: input_text.user_1_notify
  title: "Garage"
  message: "Door opened"
```

---

## Changing phones

To replace a phone:

* update `input_text.user_1_phone` from `pixel_7a` to `pixel_9`
* the automation rebuilds derived helpers automatically
* no automation refactor required

---

## Installation

1. Copy files into your config (adapt paths to your structure)

* `helpers/phones_helpers.yaml`
* `automations/phones_sync_derived_helpers.yaml`
* `scripts/mobile_notifications.yaml`
* `template_sensors/phones_bssid_dynamic.yaml` (optional)

2. Set the master helpers:

* `input_text.user_1_phone` = your phone slug (example: `pixel_7a`)
* `input_text.user_2_phone` = your phone slug

Optional:

* `input_text.user_1_name` / `input_text.user_2_name` for UI labels.

3. Reload scripts, automations, and template entities.

4. Verify derived helpers are populated:

* `input_text.user_1_notify` == `notify.mobile_app_<your_phone>`
* `input_text.user_1_tracker` == `device_tracker.<your_phone>`
* `input_text.user_1_wifi_bssid` == `sensor.<your_phone>_wi_fi_bssid`

---

## Included scripts

### `script.notification_send`

Send to a **dynamic target** (resolved via helper).

Fields:

* `target` (example: `input_text.user_1_notify`)
* `title`
* `message`

### `script.notification_send_household`

Send to both phones:

* `input_text.user_1_notify`
* `input_text.user_2_notify`

### `script.notification_send_advanced`

Send with extra Android parameters via `extra` (passed as `data:` to notify).

---

## Notes / constraints

* Assumes standard HA mobile app naming:

  * `notify.mobile_app_<phone>`
  * `device_tracker.<phone>`
  * `sensor.<phone>_wi_fi_bssid`
* If you use different entity_ids, adjust the derivation logic in the automation.

---

## License

MIT
