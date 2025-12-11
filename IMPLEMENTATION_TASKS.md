# ChillLoop QuickStart v2.0 - Implementation Task List

**Last Updated**: 2025-12-10
**Based On**: ChillLoop HomeLab installation plan 12-10-25.md
**Target System**: BeeLink SER5 (Windows 11 + WSL2) + RTX 3060 eGPU + Onn 4K Pro

---

## Overview

This task list provides a structured, phase-by-phase checklist for implementing ChillLoop QuickStart v2.0 from initial setup through production deployment. Each phase includes validation checkpoints and rollback procedures where applicable.

**Total Phases**: 15 (14 core + 1 optional Android Audio)
**Estimated Total Time**: 6-8 hours (excluding Android Audio)

---

## Phase 1: Prerequisites & Validation

**Duration**: ~15 minutes
**Goal**: Verify all hardware and network prerequisites are met

### Tasks

- [ ] **Hardware Verification**
  - [ ] Confirm BeeLink SER5 mini-PC is powered on
  - [ ] Verify RTX 3060 eGPU connected via OccuLink
  - [ ] Check TV HDMI connected to SER5 (integrated graphics), NOT eGPU
  - [ ] Confirm Onn 4K Pro is on same network

- [ ] **Windows GPU Detection**
  ```powershell
  nvidia-smi  # Should show RTX 3060
  ```
  - [ ] GPU model: RTX 3060
  - [ ] Driver version: Latest recommended
  - [ ] VRAM: 12GB

- [ ] **WSL2 GPU Support**
  ```bash
  wsl --version  # Should be >= 1.2
  wsl nvidia-smi # Should show RTX 3060
  ```
  - [ ] WSL version 1.2 or higher
  - [ ] GPU visible in WSL

- [ ] **Docker Desktop GPU Settings**
  - [ ] Open Docker Desktop → Settings → Resources
  - [ ] WSL Integration: Enable for Ubuntu distribution
  - [ ] GPU support: Enabled

- [ ] **Network Planning**
  - [ ] Reserve static IP for EC71 cameras (e.g., 192.168.1.100-104)
  - [ ] Reserve static IP for Onn 4K Pro (e.g., 192.168.1.50)
  - [ ] Document IPs in spreadsheet or note

### Validation Checkpoint

✅ All hardware powered and connected
✅ GPU visible in both Windows and WSL
✅ Docker Desktop GPU support enabled
✅ Static IPs reserved

### Troubleshooting

**GPU not visible in Windows**:
- Try HDMI dummy plug on RTX 3060
- Update NVIDIA drivers
- Check OccuLink connection

**WSL2 not installed**:
```powershell
wsl --install
# Restart computer
```

---

## Phase 2: Docker & WSL2 Setup

**Duration**: ~30 minutes
**Goal**: Install and configure Docker Desktop with WSL2 backend and GPU support

### Tasks

- [ ] **Install Docker Desktop** (if not already installed)
  - [ ] Download from https://www.docker.com/products/docker-desktop
  - [ ] Run installer
  - [ ] Select "Use WSL 2 instead of Hyper-V" during setup
  - [ ] Restart computer if prompted

- [ ] **Configure Docker Desktop**
  - [ ] Open Docker Desktop
  - [ ] Settings → General: "Use the WSL 2 based engine" checked
  - [ ] Settings → Resources → WSL Integration: Enable Ubuntu
  - [ ] Apply & Restart

- [ ] **Test Docker GPU Access**
  ```bash
  # In WSL Ubuntu
  docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
  ```
  - [ ] Command succeeds
  - [ ] Shows RTX 3060 details
  - [ ] No errors

- [ ] **Install WSL2 Dependencies**
  ```bash
  # In WSL Ubuntu
  sudo apt update
  sudo apt install -y curl jq unzip ffmpeg mosquitto-clients yamllint
  ```
  - [ ] All packages install successfully
  - [ ] No dependency errors

### Validation Checkpoint

✅ Docker Desktop running with WSL2 backend
✅ GPU accessible from Docker containers
✅ All dependencies installed in WSL

### Rollback Procedure

If Docker Desktop issues occur:
```bash
# Uninstall Docker Desktop
# Restart computer
# Reinstall with WSL2 backend option
```

---

## Phase 3: Directory & Configuration Setup

**Duration**: ~20 minutes
**Goal**: Set up project directory structure and configure environment variables

### Tasks

- [ ] **Create/Navigate to Project Directory**
  ```bash
  # In WSL Ubuntu
  cd /mnt/c/Dev/projects
  # If directory doesn't exist:
  mkdir -p /mnt/c/Dev/projects
  ```

- [ ] **Obtain ChillLoop QuickStart Project**

  **Option A: Clone from Git** (if repository exists)
  ```bash
  git clone <repository-url> /mnt/c/Dev/projects/chillloop-quickstart
  cd /mnt/c/Dev/projects/chillloop-quickstart
  ```

  **Option B: Copy from USB/Drive**
  ```bash
  cp -r /mnt/d/ChillLoop-Boot/quickstart /mnt/c/Dev/projects/chillloop-quickstart
  cd /mnt/c/Dev/projects/chillloop-quickstart
  ```

- [ ] **Copy Environment Template**
  ```bash
  cp .env.example .env
  ```

- [ ] **Configure .env File**
  ```bash
  nano .env
  # OR use VS Code: code .env
  ```

  Configure these critical variables:
  - [ ] `TZ=America/New_York` (your timezone)
  - [ ] `MQTT_USER=chillloop`
  - [ ] `MQTT_PASS=<strong-password>` (**CHANGE THIS**)
  - [ ] `EC71_USER=camera-user` (Tapo camera account)
  - [ ] `EC71_PASS=<camera-password>` (**SET THIS**)
  - [ ] `EC71_IP=192.168.1.100` (your camera IP)
  - [ ] `FRIGATE_RTSP_PASSWORD=<strong-password>` (**CHANGE THIS**)
  - [ ] `HA_NOTIFY_SERVICE=notify.mobile_app_your_phone` (if using HA Companion)
  - [ ] `HA_LATITUDE=40.7128` (your home latitude)
  - [ ] `HA_LONGITUDE=-74.0060` (your home longitude)
  - [ ] `WEBUI_SECRET_KEY=<random-secret>` (**CHANGE THIS**)
  - [ ] `ONN_IP=192.168.1.50` (Onn 4K Pro IP)
  - [ ] `HA_URL=http://localhost:8123`
  - [ ] `FRIGATE_URL=http://localhost:5000`

- [ ] **Save .env File**

- [ ] **Verify .env Not in Git** (if using git)
  ```bash
  git status
  # .env should NOT appear (it's in .gitignore)
  ```

### Validation Checkpoint

✅ Project directory exists at /mnt/c/Dev/projects/chillloop-quickstart
✅ .env file created and configured
✅ All secrets changed from defaults
✅ .env not tracked by git

### Rollback Procedure

```bash
# Delete .env and start over
rm .env
cp .env.example .env
nano .env
```

---

## Phase 4: Service Initialization

**Duration**: ~20 minutes
**Goal**: Run bootstrap script and start all Docker services

### Tasks

- [ ] **Run Bootstrap Script**
  ```bash
  cd /mnt/c/Dev/projects/chillloop-quickstart
  bash scripts/bootstrap.sh
  ```

  Expected output:
  - [ ] "All required tools found!"
  - [ ] "MQTT users created!" (chillloop, frigate, homeassistant)
  - [ ] "Frigate configuration updated!"
  - [ ] "Data directories created!"
  - [ ] "NVIDIA GPU detected!" (with GPU details)
  - [ ] No errors

- [ ] **Start All Services**
  ```bash
  bash scripts/up.sh
  ```

  Expected behavior:
  - [ ] Pulls Docker images (first run only, ~5-10 minutes)
  - [ ] Creates containers: mosquitto, homeassistant, frigate, ollama, open-webui
  - [ ] All containers start successfully

- [ ] **Verify Containers Running**
  ```bash
  docker compose -f docker-compose.yml -f docker-compose.windows.yml ps
  ```

  Expected output:
  - [ ] mosquitto: Up
  - [ ] homeassistant: Up
  - [ ] frigate: Up
  - [ ] ollama: Up
  - [ ] open-webui: Up
  - [ ] All show "healthy" status (may take 1-2 minutes)

- [ ] **Check Service Access**

  Open in browser (from Windows):
  - [ ] Home Assistant: http://localhost:8123 (shows setup wizard)
  - [ ] Frigate: http://localhost:5000 (shows dashboard)
  - [ ] Open-WebUI: http://localhost:3000 (shows login)
  - [ ] Ollama API: http://localhost:11434 (shows "Ollama is running")

### Validation Checkpoint

✅ Bootstrap completed without errors
✅ All 5 containers running
✅ All web interfaces accessible
✅ No port conflicts

### Troubleshooting

**Bootstrap fails with "docker not found"**:
- Verify Docker Desktop is running
- Check WSL integration enabled for Ubuntu

**Port conflicts (e.g., "port 8123 already in use")**:
```bash
# Find process using port
netstat -ano | findstr :8123
# Kill process or change port in docker-compose.yml
```

**Containers fail to start**:
```bash
# Check logs
bash scripts/logs.sh
# Look for specific error messages
```

### Rollback Procedure

```bash
bash scripts/down.sh
# Fix issues
# Re-run bootstrap and up
bash scripts/bootstrap.sh
bash scripts/up.sh
```

---

## Phase 5: Camera Configuration

**Duration**: ~30 minutes (per camera)
**Goal**: Enable RTSP on EC71 cameras and add to Frigate

### Tasks

- [ ] **Enable RTSP on EC71 Cameras** (for EACH camera)

  In Tapo mobile app:
  1. [ ] Open camera in Tapo app
  2. [ ] Tap gear icon (Settings)
  3. [ ] Scroll to "Advanced Settings"
  4. [ ] Tap "Camera Account"
  5. [ ] Create new account:
     - Username: `camera-user` (matches .env EC71_USER)
     - Password: `<strong-password>` (matches .env EC71_PASS)
     - **Important**: This is NOT your Tapo account
  6. [ ] Back to Advanced Settings
  7. [ ] Enable "RTSP" toggle
  8. [ ] Enable "ONVIF" toggle (optional but recommended)
  9. [ ] Note camera IP address

- [ ] **Test RTSP Streams** (from WSL)
  ```bash
  # Test sub-stream (for detection)
  ffmpeg -i "rtsp://camera-user:password@192.168.1.100:554/stream2" -frames:v 1 test_stream2.jpg

  # Test main stream (for recording)
  ffmpeg -i "rtsp://camera-user:password@192.168.1.100:554/stream1" -frames:v 1 test_stream1.jpg
  ```

  - [ ] Both streams work (creates .jpg files)
  - [ ] No authentication errors
  - [ ] No connection timeouts

- [ ] **Add Cameras to Frigate**

  **Method 1: Using Script** (Recommended for additional cameras)
  ```bash
  # First camera (if not already in config from bootstrap)
  bash scripts/add_camera.sh --name cam_front --ip 192.168.1.100

  # Additional cameras
  bash scripts/add_camera.sh --name cam_driveway --ip 192.168.1.101
  bash scripts/add_camera.sh --name cam_backyard --ip 192.168.1.102
  ```

  - [ ] Script completes without errors
  - [ ] Frigate restarts automatically

  **Method 2: Manual Config Edit** (if script unavailable)
  ```bash
  nano config/frigate/config.yml
  # Add camera blocks for each camera (see seed config for template)
  # Restart Frigate
  docker compose -f docker-compose.yml -f docker-compose.windows.yml restart frigate
  ```

- [ ] **Verify Cameras in Frigate UI**

  Open http://localhost:5000:
  - [ ] Navigate to "Cameras" page
  - [ ] All cameras appear in list
  - [ ] Live view works for each camera
  - [ ] No "No Signal" errors

### Validation Checkpoint

✅ RTSP enabled on all cameras
✅ Camera accounts created (NOT Tapo login)
✅ All streams tested successfully
✅ Cameras appear in Frigate UI with live feed

### Troubleshooting

**"401 Unauthorized" when testing RTSP**:
- Verify using camera account credentials (NOT Tapo login)
- Check username/password match .env values
- Ensure RTSP enabled in Tapo app

**"Connection timeout"**:
- Verify camera IP is correct (check router DHCP leases)
- Ping camera: `ping 192.168.1.100`
- Check camera is on same network as SER5

**Frigate shows "No Signal"**:
```bash
# Check Frigate logs
bash scripts/logs.sh frigate
# Look for RTSP connection errors
```

### Rollback Procedure

```bash
# Remove camera from config
nano config/frigate/config.yml
# Delete camera block
# Restart Frigate
docker compose -f docker-compose.yml -f docker-compose.windows.yml restart frigate
```

---

## Phase 6: Home Assistant Integration

**Duration**: ~30 minutes
**Goal**: Complete HA initial setup and integrate with Frigate

### Tasks

- [ ] **Complete Home Assistant Initial Setup**

  Open http://localhost:8123:
  1. [ ] Create admin account (username, password)
  2. [ ] Set home name
  3. [ ] Confirm location (should match HA_LATITUDE/LONGITUDE)
  4. [ ] Skip analytics if desired
  5. [ ] Finish setup wizard

- [ ] **Add Frigate Integration**

  In Home Assistant:
  1. [ ] Go to Settings → Devices & Services
  2. [ ] Click "+ ADD INTEGRATION"
  3. [ ] Search for "Frigate"
  4. [ ] Enter Frigate URL: `http://frigate:5000` (Docker network DNS)
  5. [ ] Click Submit
  6. [ ] Integration added successfully

- [ ] **Verify Camera Entities**

  - [ ] Go to Settings → Devices & Services → Frigate
  - [ ] Click on Frigate integration
  - [ ] Verify all cameras appear as entities:
    - `camera.cam_front`
    - `camera.cam_driveway`
    - `camera.cam_backyard`
    - etc.

- [ ] **Configure HA Packages** (Optional but recommended)

  Check if packages directory included:
  ```bash
  ls config/homeassistant/packages/
  ```

  If empty, copy helpers:
  ```bash
  cp android_tv/ha_android_tv_helpers.yaml config/homeassistant/packages/
  ```

  - [ ] Restart Home Assistant: Settings → System → Restart

- [ ] **Set Up Notification Services**

  **HA Companion App** (recommended):
  - [ ] Install HA Companion app on your phone (iOS/Android)
  - [ ] Sign in with your HA account
  - [ ] Allow notifications
  - [ ] Note the entity ID (e.g., `notify.mobile_app_iphone`)
  - [ ] Update `HA_NOTIFY_SERVICE` in .env if different

  **Telegram** (optional fallback):
  - [ ] Create Telegram bot (search BotFather in Telegram)
  - [ ] Get bot token
  - [ ] Get your chat ID (send message to bot, check updates)
  - [ ] Add `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` to .env
  - [ ] Restart HA

### Validation Checkpoint

✅ Home Assistant setup complete
✅ Frigate integration added
✅ All camera entities visible
✅ Notification service configured

### Troubleshooting

**Frigate integration fails "Unable to connect"**:
- Use `http://frigate:5000` (Docker network name), NOT `localhost:5000`
- Check Frigate container is running: `docker ps`
- Test from HA container:
  ```bash
  docker exec chillloop-homeassistant curl http://frigate:5000/api/version
  ```

**Camera entities not appearing**:
- Wait 1-2 minutes for HA to discover entities
- Check Frigate → Settings → MQTT is connected
- Restart Home Assistant

### Rollback Procedure

```bash
# Remove Frigate integration
# In HA UI: Settings → Devices & Services → Frigate → Delete
# Re-add integration with correct URL
```

---

## Phase 7: Android TV Setup (Onn 4K Pro)

**Duration**: ~20 minutes (Mode A) or ~40 minutes (Mode B)
**Goal**: Configure Onn 4K Pro for dashboard and camera casting

### Tasks

**Choose ONE mode**: Mode A (HA Cast) OR Mode B (Fully Kiosk)

### Mode A: HA Cast (Zero Setup)

- [ ] **Generate HA Long-Lived Access Token**

  In Home Assistant:
  1. [ ] Click profile icon (bottom left)
  2. [ ] Scroll to "Long-Lived Access Tokens"
  3. [ ] Click "CREATE TOKEN"
  4. [ ] Name: "Casting Scripts"
  5. [ ] Copy token

- [ ] **Add Token to .env**
  ```bash
  nano .env
  # Add line:
  # HA_TOKEN=<paste-token-here>
  ```

- [ ] **Test Dashboard Casting**
  ```bash
  bash scripts/cast_dashboard.sh
  ```

  - [ ] HA dashboard appears on Onn 4K Pro TV
  - [ ] No errors in script output

- [ ] **Test Camera Casting**
  ```bash
  bash scripts/cast_camera.sh cam_front
  ```

  - [ ] Camera feed appears on TV
  - [ ] No errors

- [ ] **Skip Mode B tasks** (jump to Validation Checkpoint)

### Mode B: Fully Kiosk Browser (True Kiosk)

- [ ] **Enable ADB on Onn 4K Pro**

  On Onn 4K Pro:
  1. [ ] Go to Settings → Device Preferences → About
  2. [ ] Click "Build" 7 times (until "You are now a developer" appears)
  3. [ ] Go to Settings → Device Preferences → Developer Options
  4. [ ] Enable "Network debugging"
  5. [ ] Note the IP address (should match ONN_IP in .env)

- [ ] **Install Fully Kiosk**

  **From Windows PowerShell**:
  ```powershell
  cd C:\Dev\projects\chillloop-quickstart\android_tv
  .\adb_install_fully.ps1
  # When prompted, enter Onn IP: 192.168.1.50
  ```

  OR **From WSL Bash**:
  ```bash
  cd /mnt/c/Dev/projects/chillloop-quickstart/android_tv
  bash adb_install_fully.sh
  # When prompted, enter Onn IP: 192.168.1.50
  ```

  - [ ] ADB connects successfully
  - [ ] APK installs without errors
  - [ ] Fully Kiosk opens automatically

- [ ] **Configure Fully Kiosk**

  On Onn 4K Pro (using remote):
  1. [ ] In Fully Kiosk settings:
  2. [ ] Start URL: `http://<HA_IP>:8123/lovelace/kiosk`
     - Replace `<HA_IP>` with SER5 IP (e.g., `http://192.168.1.10:8123/lovelace/kiosk`)
  3. [ ] Enable "Keep Screen On"
  4. [ ] Enable "Remote Administration"
  5. [ ] Set Remote Admin Password (use value from .env FULLY_REMOTE_ADMIN_PASSWORD)
  6. [ ] Save settings
  7. [ ] Exit settings (dashboard should load)

- [ ] **Disable ADB** (security best practice)

  On Onn 4K Pro:
  - [ ] Settings → Device Preferences → Developer Options → Network debugging: **OFF**

### Validation Checkpoint

**Mode A (HA Cast)**:
✅ HA token generated and added to .env
✅ cast_dashboard.sh works
✅ Dashboard appears on TV

**Mode B (Fully Kiosk)**:
✅ Fully Kiosk installed on Onn
✅ Start URL configured
✅ Dashboard loads automatically
✅ ADB disabled

### Troubleshooting

**HA Cast: "No cast devices found"**:
- Ensure Onn and SER5 on same network
- Check Google Cast integration in HA (auto-discovers)
- Restart Onn 4K Pro

**Fully Kiosk: ADB connection refused**:
- Verify Network debugging enabled
- Check Onn IP is correct
- Try `adb connect 192.168.1.50:5555` manually

**Fully Kiosk: Dashboard won't load**:
- Check Start URL uses IP (not localhost)
- Verify HA is accessible from Onn: Open Chrome on Onn, navigate to `http://<SER5_IP>:8123`

### Rollback Procedure

**Mode B only**:
```bash
# Uninstall Fully Kiosk
adb connect 192.168.1.50:5555
adb uninstall de.ozerov.fully
```

---

## Phase 8: GPU & Ollama Validation

**Duration**: ~20 minutes
**Goal**: Verify GPU passthrough and Ollama functionality

### Tasks

- [ ] **Verify Ollama Container Using GPU**
  ```bash
  nvidia-smi -l 1
  # Watch for "ollama" process in process list
  ```

  - [ ] Wait 30 seconds to 1 minute
  - [ ] "ollama" appears in GPU process list
  - [ ] GPU memory usage shows allocation

- [ ] **Pull Initial Ollama Models**

  **Via Open-WebUI** (recommended):
  1. [ ] Open http://localhost:3000
  2. [ ] Create account (first user is admin)
  3. [ ] Click "Download Model" or "+"
  4. [ ] Search for "llama2" or "mistral"
  5. [ ] Click "Download"
  6. [ ] Wait for download to complete (~5-10 minutes for 7B model)

  OR **Via CLI**:
  ```bash
  docker exec -it chillloop-ollama ollama pull llama2
  ```

- [ ] **Test Ollama API**
  ```bash
  curl http://localhost:11434/api/tags | jq .
  ```

  Expected output:
  - [ ] JSON response
  - [ ] Shows downloaded models in `models` array
  - [ ] No errors

- [ ] **Test Model Inference**

  In Open-WebUI:
  - [ ] Select downloaded model
  - [ ] Send test prompt: "Hello, how are you?"
  - [ ] Receive response within reasonable time (~5-10 seconds for 7B model)
  - [ ] Response quality is coherent

- [ ] **Monitor GPU Usage During Inference**
  ```bash
  nvidia-smi
  ```

  - [ ] GPU memory usage increases during inference
  - [ ] GPU utilization shows activity
  - [ ] No errors or warnings

### Validation Checkpoint

✅ Ollama container using GPU (visible in nvidia-smi)
✅ At least one model pulled successfully
✅ API responds to queries
✅ Inference works in Open-WebUI
✅ GPU usage increases during inference

### Troubleshooting

**Ollama not using GPU**:
- Check docker-compose.windows.yml has GPU config:
  ```yaml
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            capabilities: [gpu]
  ```
- Restart Ollama container:
  ```bash
  docker compose -f docker-compose.yml -f docker-compose.windows.yml restart ollama
  ```

**Model download fails**:
- Check internet connection
- Try different model (some are very large)
- Check disk space: `df -h`

**Inference very slow (>30s)**:
- Verify GPU is being used: `nvidia-smi`
- Try smaller model (7B vs 13B)
- Check GPU VRAM not exhausted

### Rollback Procedure

```bash
# Remove downloaded models
docker exec -it chillloop-ollama ollama rm <model-name>

# Restart Ollama
docker compose -f docker-compose.yml -f docker-compose.windows.yml restart ollama
```

---

## Phase 9: MQTT Testing & Validation

**Duration**: ~15 minutes
**Goal**: Verify MQTT broker functionality and message flow

### Tasks

- [ ] **Test MQTT Pub/Sub**

  **Terminal 1** (Subscribe):
  ```bash
  mosquitto_sub -h localhost -u chillloop -P <MQTT_PASS> -t 'home/#' -v
  ```

  **Terminal 2** (Publish):
  ```bash
  mosquitto_pub -h localhost -u chillloop -P <MQTT_PASS> -t 'home/test' -m 'Hello MQTT'
  ```

  - [ ] Terminal 1 shows: `home/test Hello MQTT`
  - [ ] No authentication errors
  - [ ] No connection refused

- [ ] **Verify Frigate Publishing to MQTT**

  **Terminal 1** (Subscribe to Frigate events):
  ```bash
  mosquitto_sub -h localhost -u chillloop -P <MQTT_PASS> -t 'frigate/#' -v
  ```

  **Trigger motion** (walk in front of camera or wave hand):

  - [ ] Messages appear in Terminal 1
  - [ ] Format: `frigate/events` with JSON payload
  - [ ] Contains camera name, label (person/car), event_id

- [ ] **Verify HA Receiving MQTT Messages**

  In Home Assistant:
  1. [ ] Developer Tools → MQTT
  2. [ ] Subscribe to topic: `frigate/events`
  3. [ ] Trigger camera motion
  4. [ ] Messages appear in HA MQTT panel
  5. [ ] Payload can be expanded and read

- [ ] **Test MQTT Sensors in HA** (if configured)

  ```bash
  mosquitto_pub -h localhost -u chillloop -P <MQTT_PASS> -t 'home/sensors/test_sensor' -m 'ON'
  ```

  In Home Assistant:
  - [ ] Check Developer Tools → States
  - [ ] Search for `sensor.test_sensor` or binary_sensor
  - [ ] State updates to "ON"

### Validation Checkpoint

✅ MQTT pub/sub works with credentials
✅ Frigate publishes events to MQTT
✅ Home Assistant receives MQTT messages
✅ Test sensors respond to MQTT messages

### Troubleshooting

**"Connection refused"**:
- Check Mosquitto container running: `docker ps | grep mosquitto`
- Verify MQTT port 1883 not blocked
- Check logs: `bash scripts/logs.sh mosquitto`

**"Authentication failed"**:
- Verify password matches .env MQTT_PASS
- Re-run bootstrap to regenerate password file:
  ```bash
  bash scripts/bootstrap.sh
  docker compose -f docker-compose.yml -f docker-compose.windows.yml restart mosquitto
  ```

**No Frigate events**:
- Check Frigate logs: `bash scripts/logs.sh frigate`
- Verify MQTT configured in config/frigate/config.yml:
  ```yaml
  mqtt:
    enabled: true
    host: mosquitto
  ```

### Rollback Procedure

```bash
# Regenerate MQTT passwords
bash scripts/bootstrap.sh
docker compose -f docker-compose.yml -f docker-compose.windows.yml restart mosquitto
```

---

## Phase 10: Automation & Testing

**Duration**: ~30 minutes
**Goal**: Configure and test motion detection automations

### Tasks

- [ ] **Configure Motion Detection Automation**

  Create or edit: `config/homeassistant/packages/camera_motion.yaml`

  ```yaml
  automation:
    - alias: "Camera: Person Detected"
      trigger:
        - platform: mqtt
          topic: "frigate/events"
      condition:
        - "{{ trigger.payload_json.type == 'new' }}"
        - "{{ trigger.payload_json.after.label == 'person' }}"
      action:
        - service: notify.mobile_app_your_phone
          data:
            title: "Person Detected"
            message: "{{ trigger.payload_json.after.camera }} camera"
  ```

  - [ ] Save file
  - [ ] Reload automations in HA: Developer Tools → YAML → Automations

- [ ] **Test Person Detection Notification**

  1. [ ] Walk in front of camera
  2. [ ] Wait 5-10 seconds
  3. [ ] Check phone for notification
  4. [ ] Notification received with camera name

- [ ] **Test Camera Clip Casting to TV** (if using HA Cast)

  Create automation to cast on detection:
  ```yaml
  automation:
    - alias: "Cast: Person at Front Door"
      trigger:
        - platform: mqtt
          topic: "frigate/events"
      condition:
        - "{{ trigger.payload_json.after.camera == 'cam_front' }}"
        - "{{ trigger.payload_json.after.label == 'person' }}"
      action:
        - service: script.cast_frigate_event
  ```

  - [ ] Trigger event at front camera
  - [ ] Clip casts to TV
  - [ ] Shows snapshot or clip

- [ ] **Configure Camera Offline Alerts**

  ```yaml
  automation:
    - alias: "Alert: Camera Offline"
      trigger:
        - platform: state
          entity_id:
            - camera.cam_front
            - camera.cam_driveway
          to: "unavailable"
          for: "00:05:00"  # 5 minutes
      action:
        - service: notify.mobile_app_your_phone
          data:
            title: "Camera Offline"
            message: "{{ trigger.entity_id }} is unavailable"
  ```

  - [ ] Test by unplugging camera
  - [ ] Wait 5 minutes
  - [ ] Receive offline notification

- [ ] **Test TTS Announcements** (optional)

  ```yaml
  automation:
    - alias: "Announce: Person at Door"
      trigger:
        - platform: mqtt
          topic: "frigate/events"
      action:
        - service: tts.google_translate_say
          entity_id: media_player.onn_4k_pro
          data:
            message: "Person detected at front door"
  ```

  - [ ] Trigger event
  - [ ] TTS plays on TV speakers

### Validation Checkpoint

✅ Motion detection automation works
✅ Notifications received on phone
✅ Camera clip casting works (if configured)
✅ Offline alerts configured and tested
✅ TTS announcements work (if configured)

### Troubleshooting

**Automation not triggering**:
- Check automation enabled in HA UI
- View automation traces: Settings → Automations → [Automation] → Traces
- Check MQTT events: Developer Tools → MQTT → Subscribe to `frigate/events`

**Notifications not received**:
- Verify HA Companion app installed and logged in
- Check notification permissions on phone
- Test notification manually:
  ```yaml
  # Developer Tools → Services
  service: notify.mobile_app_your_phone
  data:
    message: "Test notification"
  ```

**Casting fails**:
- Verify HA_TOKEN in .env
- Check Onn 4K Pro on same network
- Test cast manually: `bash scripts/cast_dashboard.sh`

### Rollback Procedure

```bash
# Disable automations
# In HA UI: Settings → Automations → Toggle off
# OR delete automation files:
rm config/homeassistant/packages/camera_motion.yaml
# Reload HA
```

---

## Phase 11: Monitoring & Maintenance Setup

**Duration**: ~20 minutes
**Goal**: Configure backup strategy and maintenance procedures

### Tasks

- [ ] **Configure Backup Strategy**

  **What to backup**:
  - [ ] `.env` file (encrypted storage)
  - [ ] `config/` directory (all configurations)
  - [ ] `data/homeassistant/` (HA database and history)
  - [ ] Optional: `data/frigate/` (recordings - very large)

- [ ] **Create Backup Script** (if not exists)

  Create `backups/backup.sh`:
  ```bash
  #!/usr/bin/env bash
  set -euo pipefail

  BACKUP_DIR="/mnt/c/Dev/backups/chillloop"
  DATE=$(date +%Y%m%d_%H%M%S)

  mkdir -p "$BACKUP_DIR"

  # Backup configs
  tar -czf "$BACKUP_DIR/config_$DATE.tar.gz" -C /mnt/c/Dev/projects/chillloop-quickstart config/

  # Backup .env
  cp /mnt/c/Dev/projects/chillloop-quickstart/.env "$BACKUP_DIR/env_$DATE.backup"

  # Backup HA data
  tar -czf "$BACKUP_DIR/ha_data_$DATE.tar.gz" -C /mnt/c/Dev/projects/chillloop-quickstart data/homeassistant/

  echo "Backup completed: $BACKUP_DIR"
  ```

  - [ ] Make executable: `chmod +x backups/backup.sh`

- [ ] **Test Backup**
  ```bash
  bash backups/backup.sh
  ```

  - [ ] Backup completes successfully
  - [ ] Files created in backup directory
  - [ ] Archives can be extracted

- [ ] **Document Backup Schedule**

  Create `backups/README.md`:
  - [ ] Recommended frequency: Weekly
  - [ ] What to backup (list from above)
  - [ ] Backup location
  - [ ] Retention policy (e.g., keep last 4 weekly backups)

- [ ] **Set Up Log Rotation** (optional)

  If logs grow large:
  ```bash
  # Check log sizes
  docker compose -f docker-compose.yml -f docker-compose.windows.yml logs --tail=0 | wc -l
  ```

  Configure logrotate or Docker logging driver if needed.

- [ ] **Document Update Procedures**

  Create `MAINTENANCE.md`:
  - [ ] How to update services: `bash scripts/update.sh`
  - [ ] How to update Docker images
  - [ ] How to update HA integrations
  - [ ] How to update Ollama models

- [ ] **Create Restore Procedure Documentation**

  In `backups/RESTORE.md`:
  ```markdown
  # Restore Procedure

  1. Stop services: `bash scripts/down.sh`
  2. Extract config backup: `tar -xzf config_YYYYMMDD_HHMMSS.tar.gz -C /mnt/c/Dev/projects/chillloop-quickstart/`
  3. Restore .env: `cp env_YYYYMMDD_HHMMSS.backup /mnt/c/Dev/projects/chillloop-quickstart/.env`
  4. Extract HA data: `tar -xzf ha_data_YYYYMMDD_HHMMSS.tar.gz -C /mnt/c/Dev/projects/chillloop-quickstart/`
  5. Start services: `bash scripts/up.sh`
  ```

### Validation Checkpoint

✅ Backup strategy documented
✅ Backup script created and tested
✅ Backup location established
✅ Update procedures documented
✅ Restore procedure documented

### Rollback Procedure

N/A (this phase is documentation-only)

---

## Phase 12: Security Hardening

**Duration**: ~15 minutes
**Goal**: Secure the deployment by changing defaults and configuring firewall

### Tasks

- [ ] **Change All Default Passwords**

  Verify in `.env`:
  - [ ] MQTT_PASS: Changed from default
  - [ ] EC71_PASS: Camera account password set
  - [ ] FRIGATE_RTSP_PASSWORD: Changed from default
  - [ ] WEBUI_SECRET_KEY: Random string set
  - [ ] FULLY_REMOTE_ADMIN_PASSWORD: Changed from default (if using Fully)

- [ ] **Generate Home Assistant Long-Lived Access Token**

  (If not done in Phase 7)
  - [ ] In HA: Profile → Security → Long-Lived Access Tokens
  - [ ] Create token named "Casting Scripts"
  - [ ] Copy token to .env as HA_TOKEN

- [ ] **Configure Fully Kiosk Security** (if using Mode B)

  On Onn 4K Pro:
  - [ ] Remote Admin: Enable **ONLY on LAN**
  - [ ] Remote Admin Password: Strong password from .env
  - [ ] Disable "Allow Remote Config Download"

- [ ] **Document Firewall Rules** (Windows Firewall)

  Recommended rules:
  ```markdown
  ## Firewall Rules

  Block external access to:
  - Port 1883 (MQTT) - LAN only
  - Port 5000 (Frigate) - LAN only
  - Port 11434 (Ollama) - LAN only

  Allow external access:
  - Port 8123 (Home Assistant) - Only if using Nabu Casa or reverse proxy with SSL
  ```

  - [ ] Document in `SECURITY.md`

- [ ] **Verify .env Not Committed**

  ```bash
  git status
  # Should NOT show .env

  cat .gitignore | grep .env
  # Should show: .env
  ```

- [ ] **Set File Permissions** (WSL)

  ```bash
  chmod 600 .env
  chmod 600 config/mosquitto/passwd
  ```

### Validation Checkpoint

✅ All default passwords changed
✅ HA token generated
✅ Fully Kiosk secured (if used)
✅ Firewall rules documented
✅ .env not in git
✅ Sensitive files have restricted permissions

### Troubleshooting

**Forgot to change password**:
- Edit .env
- Re-run bootstrap: `bash scripts/bootstrap.sh`
- Restart affected services

**Can't access HA from outside network**:
- Recommended: Use Nabu Casa (https://www.nabucasa.com/)
- OR set up reverse proxy with Let's Encrypt SSL

---

## Phase 13: Performance Tuning

**Duration**: ~20 minutes
**Goal**: Optimize Frigate detection, storage, and network usage

### Tasks

- [ ] **Adjust Frigate Detection Settings**

  Edit `config/frigate/config.yml`:

  **For CPU detection** (current):
  ```yaml
  detectors:
    cpu1:
      type: cpu
      num_threads: 3  # Adjust based on CPU load
  ```

  - [ ] Monitor CPU usage: `top` or Task Manager
  - [ ] Adjust num_threads if CPU overloaded (reduce) or underutilized (increase)

  **For future TensorRT** (when ready):
  ```yaml
  detectors:
    tensorrt:
      type: tensorrt
      device: 0  # RTX 3060

  ffmpeg:
    hwaccel_args: preset-nvidia-h264
  ```

- [ ] **Configure Recording Retention**

  In `config/frigate/config.yml`:
  ```yaml
  record:
    enabled: true
    retain:
      days: 7  # Adjust based on available storage
      mode: motion  # Only record motion segments
  ```

  **Storage calculation**:
  - 1 camera @ 1080p ≈ 2-5GB/day
  - 4 cameras × 7 days ≈ 60-140GB

  - [ ] Check available storage: `df -h /mnt/c/Dev/projects/chillloop-quickstart/data/frigate`
  - [ ] Adjust `days` if needed

- [ ] **Monitor GPU Usage During Inference**

  ```bash
  nvidia-smi -l 1
  ```

  - [ ] Check GPU memory usage
  - [ ] Check GPU utilization (should be >50% during active inference)
  - [ ] If underutilized, consider pulling larger Ollama model

- [ ] **Optimize Camera Streams**

  Verify dual-stream configuration:
  - [ ] Sub-stream (stream2) used for detection: Lower resolution (720p), lower bitrate
  - [ ] Main stream (stream1) used for recording: Higher resolution (1080p), higher quality

  In Frigate config:
  ```yaml
  cameras:
    cam_front:
      ffmpeg:
        inputs:
          - path: rtsp://.../stream2  # 720p for detect
            roles: [detect]
          - path: rtsp://.../stream1  # 1080p for record
            roles: [record]
  ```

- [ ] **Test Frigate Performance**

  In Frigate UI (http://localhost:5000):
  - [ ] Go to System stats
  - [ ] Check detection FPS (should be 5-10 FPS)
  - [ ] Check inference speed (should be <100ms for CPU)
  - [ ] Check for dropped frames (should be minimal)

### Validation Checkpoint

✅ Frigate detection settings optimized
✅ Recording retention configured for available storage
✅ GPU usage monitored and efficient
✅ Camera streams using dual-stream configuration
✅ Frigate performance metrics acceptable

### Troubleshooting

**High CPU usage**:
- Reduce num_threads in detector config
- Lower detection FPS: `detect: { fps: 3 }`
- Use sub-stream for all cameras

**Disk filling up quickly**:
- Reduce retention days
- Enable motion-only recording
- Check snapshot retention: `snapshots: { retain: { default: 3 } }`

**Detection too slow**:
- Increase num_threads (if CPU available)
- Consider TensorRT once ready
- Reduce camera resolution

---

## Phase 14: Documentation & Handoff

**Duration**: ~30 minutes
**Goal**: Create comprehensive documentation for ongoing operations

### Tasks

- [ ] **Document Custom Configurations**

  Create `CUSTOM_CONFIG.md`:
  - [ ] List all cameras with names and locations
  - [ ] Custom automations created
  - [ ] Modified settings from defaults
  - [ ] Ollama models installed
  - [ ] HA integrations added

- [ ] **Create Troubleshooting Runbook**

  Create `TROUBLESHOOTING.md` with sections:
  - [ ] "Services won't start" → Check Docker Desktop, logs
  - [ ] "Camera offline" → Check RTSP, IP, power
  - [ ] "GPU not working" → nvidia-smi, Docker GPU config
  - [ ] "Notifications not received" → Check HA Companion, test service
  - [ ] "Casting fails" → Check token, network, Google Cast integration

- [ ] **Document Camera Locations and Names**

  Create `CAMERAS.md`:
  ```markdown
  | Camera Name | Location | IP Address | RTSP URL |
  |-------------|----------|------------|----------|
  | cam_front | Front Door | 192.168.1.100 | rtsp://... |
  | cam_driveway | Driveway | 192.168.1.101 | rtsp://... |
  ```

- [ ] **Create Spouse-Friendly Operation Guide**

  Create `QUICK_START_USER.md`:
  ```markdown
  # ChillLoop User Guide

  ## Viewing Cameras
  1. Open Home Assistant on phone or TV
  2. Go to "Cameras" tab
  3. Tap camera to view live feed

  ## Casting to TV
  - Say "Hey Google, show front door camera" (if using Google Home)
  - OR open HA app, tap camera, tap cast icon

  ## Checking Recordings
  1. Open Frigate: http://localhost:5000
  2. Click "Events" tab
  3. Browse by date and camera
  ```

- [ ] **Document Common Maintenance Tasks**

  In `MAINTENANCE.md`:
  ```markdown
  ## Weekly Tasks
  - [ ] Run backup: `bash backups/backup.sh`
  - [ ] Check storage: `df -h`

  ## Monthly Tasks
  - [ ] Update services: `bash scripts/update.sh`
  - [ ] Check for HA updates
  - [ ] Review Frigate recordings, adjust retention if needed

  ## Quarterly Tasks
  - [ ] Review and clean old recordings
  - [ ] Update Ollama models if new versions available
  - [ ] Test restore procedure
  ```

- [ ] **Create FAQ Document**

  Create `FAQ.md` with common questions:
  - How do I add a new camera?
  - How do I change passwords?
  - How do I access when away from home?
  - What if power goes out?

### Validation Checkpoint

✅ Custom configurations documented
✅ Troubleshooting runbook created
✅ Camera locations documented
✅ User-friendly guide created
✅ Maintenance tasks documented
✅ FAQ created

### Rollback Procedure

N/A (documentation-only phase)

---

## Phase 15: Android Audio Node Setup (Optional - Post-Christmas)

**Duration**: ~2 hours (includes APK build, testing, and deployment to 2 devices)
**Goal**: Deploy Android Audio monitoring nodes on Samsung S10e and Note20

**⚠️ NOTE**: This is an OPTIONAL phase for post-Christmas deployment. The core ChillLoop system (Phases 1-14) is complete without this phase.

### Tasks

- [ ] **Review Android Audio Documentation**

  - [ ] Read: `chillloop-android-audio-SSoT-v0.2.0.md`
  - [ ] Understand architecture (Foreground Service, RMS analysis, MQTT publishing)
  - [ ] Note dependencies (Kotlin Coroutines, Paho MQTT, AndroidX)
  - [ ] Review security model (BuildConfig for MVP, Keystore for production)

- [ ] **Set Up Android Development Environment** (if not already set up)

  - [ ] Install Android Studio (https://developer.android.com/studio)
  - [ ] Install Android SDK
  - [ ] Configure SDK paths
  - [ ] Install build tools

- [ ] **Build Android APK**

  1. [ ] Open Android Audio project in Android Studio
  2. [ ] Edit `gradle.properties` (gitignored):
     ```properties
     MQTT_HOST=mqtt.home
     MQTT_PORT=1883
     MQTT_USER=chillloop_mobile
     MQTT_PASSWORD=<YOUR_MQTT_PASSWORD>
     ```
  3. [ ] Build → Generate Signed Bundle / APK
  4. [ ] Select APK, Release variant
  5. [ ] Create or use existing keystore
  6. [ ] Build completes successfully

- [ ] **Configure MQTT User** (if not exists)

  ```bash
  # In WSL
  cd /mnt/c/Dev/projects/chillloop-quickstart
  mosquitto_passwd -b config/mosquitto/passwd chillloop_mobile <PASSWORD>
  docker compose -f docker-compose.yml -f docker-compose.windows.yml restart mosquitto
  ```

- [ ] **Install on Samsung S10e (Living Room Node)**

  1. [ ] Enable USB debugging on S10e
  2. [ ] Connect S10e to PC via USB
  3. [ ] Install APK:
     ```bash
     adb install -r app/build/outputs/apk/release/app-release.apk
     ```
  4. [ ] Open ChillLoop Audio app
  5. [ ] Verify `room_id` is set to "living_room" (or edit `loadConfig()` in code)
  6. [ ] Start monitoring service

- [ ] **Configure S10e Device**

  - [ ] Settings → Apps → ChillLoop → Battery → **Unrestricted**
  - [ ] Disable "Put app to sleep"
  - [ ] Open Recents, tap app icon, select "Lock" or "Pin"
  - [ ] Plug into wall charger
  - [ ] Verify notification shows "Monitoring: living_room"

- [ ] **Test MQTT Connectivity (S10e)**

  ```bash
  # Subscribe to living room metrics
  mosquitto_sub -h mqtt.home -u chillloop -P <PASSWORD> -t "chillloop/audio/living_room/metrics" -v
  ```

  - [ ] Messages appear every 500ms when sound detected
  - [ ] Format: `{"node_id":"android_s10e_living","room_id":"living_room","db_spl":"63.4","ts":1702409823456}`

- [ ] **Test Audio Publishing (S10e)**

  1. [ ] Speak loudly near S10e (normal conversation, >60dB)
  2. [ ] Check MQTT messages show `db_spl >50`
  3. [ ] Stop speaking, wait 10 seconds
  4. [ ] Check heartbeat message appears with `status="silent"`

- [ ] **Install on Note20 (Kitchen Node)**

  1. [ ] Repeat APK install process for Note20
  2. [ ] Verify `room_id` is set to "kitchen"
  3. [ ] Configure battery optimization and pinning
  4. [ ] Start monitoring service

- [ ] **Test MQTT Connectivity (Note20)**

  ```bash
  mosquitto_sub -h mqtt.home -u chillloop -P <PASSWORD> -t "chillloop/audio/kitchen/metrics" -v
  ```

  - [ ] Messages appear from Note20
  - [ ] `room_id` shows "kitchen"

- [ ] **Configure Home Assistant MQTT Sensors**

  Create `config/homeassistant/packages/audio_nodes.yaml`:
  ```yaml
  mqtt:
    sensor:
      - name: "Living Room Audio Level"
        state_topic: "chillloop/audio/living_room/metrics"
        value_template: "{{ value_json.db_spl }}"
        unit_of_measurement: "dB SPL"
        device_class: sound_pressure

      - name: "Kitchen Audio Level"
        state_topic: "chillloop/audio/kitchen/metrics"
        value_template: "{{ value_json.db_spl }}"
        unit_of_measurement: "dB SPL"
        device_class: sound_pressure
  ```

  - [ ] Save file
  - [ ] Reload YAML in HA

- [ ] **Verify Sensors in Home Assistant**

  - [ ] Go to Developer Tools → States
  - [ ] Search for "Living Room Audio Level"
  - [ ] Search for "Kitchen Audio Level"
  - [ ] Both sensors show real-time dB SPL values

- [ ] **Define Escalation Detection Rules** (Example)

  Add to audio_nodes.yaml:
  ```yaml
  automation:
    - alias: "Audio: Escalation Detected"
      trigger:
        - platform: numeric_state
          entity_id: sensor.living_room_audio_level
          above: 75
          for: "00:00:05"  # 5 seconds sustained
      action:
        - service: notify.mobile_app_parent_phone
          data:
            title: "Escalation Detected"
            message: "High noise level in living room ({{ states('sensor.living_room_audio_level') }} dB)"
            data:
              tag: "audio_escalation_living_room"
  ```

- [ ] **Test Escalation Detection**

  1. [ ] Make loud sustained noise in living room (>75dB for 5+ seconds)
  2. [ ] Wait for notification
  3. [ ] Verify notification received with correct room name

- [ ] **Test Bedtime/De-escalation Behavior Loops** (Example)

  Create automation for bedtime consistency:
  ```yaml
  automation:
    - alias: "Bedtime: Quiet Time Reminder"
      trigger:
        - platform: time
          at: "20:00:00"  # 8 PM
      condition:
        - condition: numeric_state
          entity_id: sensor.living_room_audio_level
          above: 60  # If still noisy
      action:
        - service: tts.google_translate_say
          entity_id: media_player.onn_4k_pro
          data:
            message: "It's bedtime. Time to quiet down."
        - service: notify.mobile_app_kids_phone
          data:
            message: "Bedtime in 30 minutes"
  ```

  - [ ] Test automation triggers at 8 PM
  - [ ] TTS announces reminder if room is noisy

### Validation Checkpoint

✅ Android Audio APK built successfully
✅ MQTT user `chillloop_mobile` created
✅ S10e installed and publishing metrics
✅ Note20 installed and publishing metrics
✅ HA sensors created and updating
✅ Escalation detection automation works
✅ Bedtime automation tested

### Troubleshooting

**APK build fails**:
- Check Android Studio version (latest stable)
- Verify SDK installed
- Check gradle.properties has correct values

**MQTT connection fails from Android**:
- Test DNS: `ping mqtt.home` from Android (use terminal app)
- Verify MQTT credentials in gradle.properties
- Check firewall not blocking port 1883

**Battery optimization kills service**:
- Double-check Settings → Apps → ChillLoop → Battery → Unrestricted
- Remove from "Sleeping apps" list if present
- Ensure app is pinned in Recents

**No audio metrics publishing**:
- Check RECORD_AUDIO permission granted
- Check service notification visible in status bar
- Review app logs: `adb logcat | grep ChillLoop`

### Rollback Procedure

```bash
# Uninstall app from devices
adb uninstall com.chillloop.audio

# Remove MQTT user
# Edit config/mosquitto/passwd, remove chillloop_mobile line
docker compose -f docker-compose.yml -f docker-compose.windows.yml restart mosquitto

# Remove HA sensors
rm config/homeassistant/packages/audio_nodes.yaml
# Reload HA YAML
```

---

## Implementation Complete! 🎉

### Summary of Deliverables

✅ **Phase 1-4**: Core infrastructure deployed (Docker, WSL2, GPU, services)
✅ **Phase 5-6**: Cameras configured and integrated with Home Assistant
✅ **Phase 7**: Android TV (Onn 4K Pro) set up for dashboard/casting
✅ **Phase 8-9**: Ollama LLM and MQTT broker validated
✅ **Phase 10**: Automations configured and tested
✅ **Phase 11-12**: Backup, maintenance, and security procedures established
✅ **Phase 13-14**: Performance tuned and documentation completed
✅ **Phase 15** (Optional): Android Audio nodes deployed for behavioral monitoring

### Next Steps

**Daily Operations**:
- Monitor cameras via HA app or Frigate UI
- Check notifications for motion detection
- Cast camera feeds to TV when needed

**Weekly Maintenance**:
- Run backup: `bash backups/backup.sh`
- Check storage usage: `df -h`
- Review Frigate events for any issues

**Monthly Maintenance**:
- Update services: `bash scripts/update.sh`
- Check for Home Assistant updates
- Review and adjust automations as needed

**Future Enhancements**:
- Migrate Frigate to TensorRT detection (GPU-accelerated)
- Implement uns.json service discovery (Android Audio)
- Add wake-word detection (Porcupine SDK)
- Expand behavioral loops for family dynamics

### Support Resources

- **Project Documentation**: [README.md](file:///C:/Dev/projects/chillloop-quickstart/README.md)
- **Claude Code Guidance**: [CLAUDE.md](file:///C:/Dev/projects/chillloop-quickstart/CLAUDE.md)
- **Quick Reference**: [QUICK_REFERENCE.md](file:///C:/Dev/projects/QUICK_REFERENCE.md)
- **Troubleshooting**: [TROUBLESHOOTING.md](file:///C:/Dev/projects/chillloop-quickstart/TROUBLESHOOTING.md)
- **Maintenance**: [MAINTENANCE.md](file:///C:/Dev/projects/chillloop-quickstart/MAINTENANCE.md)

### Community & Help

- Home Assistant Community: https://community.home-assistant.io/
- Frigate Documentation: https://docs.frigate.video/
- Ollama GitHub: https://github.com/ollama/ollama

---

**END OF IMPLEMENTATION TASK LIST**
