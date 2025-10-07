# SensorSync Pro - Mobile App

A cross-platform mobile application (iOS & Android) for scanning Inovonics and Milesight sensor QR/Data Matrix codes using the device camera, with real-time scanning and cloud MySQL database integration.

## 🎯 Features

### Core Functionality
- **Authentication System**: Secure login with "Remember Me" functionality
- **Site Management**: Create, view, search, filter, and delete sites
- **Dual Sensor Support**:
  - Inovonics sensors (Data Matrix codes)
  - Milesight sensors (QR codes with DevEUI)
- **Advanced Camera Scanner**:
  - Real-time barcode scanning
  - Torch/flashlight toggle
  - Haptic feedback on successful scans
  - Visual status indicators
  - 10-second cooldown between scans
  - Duplicate prevention
- **BACnet Values**: Auto-generated for Milesight sensors
- **Offline Support**: Local caching with sync when online
- **CSV Export**: Export sensor data with native share functionality

### User Interface
- **Material Design** (Android) / **Cupertino** (iOS) components
- Three-tab sensor management layout
- Pull-to-refresh on all lists
- Search and filter capabilities
- Empty states with helpful messages
- Loading indicators and error handling

## 🚀 Getting Started

### Prerequisites
- Node.js (v16 or higher)
- npm or yarn
- Expo CLI
- iOS Simulator (Mac) or Android Emulator

### Installation

1. **Clone the repository**
   ```bash
   cd frontend
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Configure API endpoint**
   Update the API URL in `src/constants/config.js`:
   ```javascript
   export const API_BASE_URL = 'http://YOUR_SERVER_IP:4000';
   ```

4. **Start the development server**
   ```bash
   npm start
   ```

5. **Run on a device**
   - **iOS**: Press `i` or scan QR code with Expo Go app
   - **Android**: Press `a` or scan QR code with Expo Go app
   - **Web**: Press `w` (limited functionality)

### Backend Setup

1. **Navigate to backend directory**
   ```bash
   cd backend
   ```

2. **Create `.env` file**
   ```env
   DB_HOST=107.161.22.238
   DB_PORT=3306
   DB_USER=root
   DB_PASSWORD=your_password
   DB_NAME=DualFuelSensors
   PORT=4000
   ```

3. **Install backend dependencies**
   ```bash
   npm install
   ```

4. **Start the backend server**
   ```bash
   npm run dev
   ```

## 📱 App Structure

```
frontend/
├── App.js                          # App entry point
├── src/
│   ├── components/                 # Reusable components
│   │   ├── BACnetTab.js           # BACnet values tab
│   │   ├── InovonicsTab.js        # Inovonics sensors tab
│   │   └── MilesightTab.js        # Milesight sensors tab
│   ├── constants/                  # App constants
│   │   ├── colors.js              # Color palette
│   │   ├── config.js              # Configuration
│   │   └── index.js               # Exports
│   ├── context/                    # React context
│   │   └── AuthContext.js         # Authentication context
│   ├── navigation/                 # Navigation setup
│   │   └── AppNavigator.js        # Main navigator
│   ├── screens/                    # Screen components
│   │   ├── AddSiteScreen.js       # Add new site
│   │   ├── LoginScreen.js         # Login/auth
│   │   ├── ScannerScreen.js       # Camera scanner
│   │   ├── SensorManagementScreen.js  # Sensor tabs
│   │   └── SitesListScreen.js     # Sites list
│   ├── services/                   # API & cache services
│   │   ├── api.js                 # API client
│   │   └── cache.js               # Offline cache
│   └── utils/                      # Utility functions
│       ├── formatters.js          # Data formatters
│       └── validation.js          # Validation logic
└── package.json
```

## 🔧 API Endpoints

### Sites
- `GET /api/sites?search=...` - Get all sites (with search)
- `GET /api/sites/:id` - Get site by ID
- `POST /api/sites` - Create new site
- `PUT /api/sites/:id` - Update site
- `DELETE /api/sites/:id` - Delete site

### Inovonics Sensors
- `GET /api/inovonics/sensors?site_id=X` - Get sensors for site
- `GET /api/inovonics/sensors/:id` - Get sensor by ID
- `POST /api/inovonics/scans` - Scan and create sensor
- `PUT /api/inovonics/sensors/:id` - Update sensor
- `DELETE /api/inovonics/sensors/:id` - Delete sensor

### Milesight Sensors
- `GET /api/milesight/sensors?site_id=X` - Get sensors for site
- `GET /api/milesight/sensors/:id` - Get sensor by ID
- `POST /api/milesight/scans` - Scan and create sensor (auto-creates BACnet values)
- `PUT /api/milesight/sensors/:id` - Update sensor
- `DELETE /api/milesight/sensors/:id` - Soft delete sensor

### BACnet Values
- `GET /api/bacnet?site_id=X&sensor_id=Y` - Get BACnet values

## 📊 Database Schema

### Sites Table
```sql
CREATE TABLE sites (
  site_id INT AUTO_INCREMENT PRIMARY KEY,
  site_name VARCHAR(255) NOT NULL,
  site_status ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE'
);
```

### Inovonics Sensors Table
```sql
CREATE TABLE inovonics_sensors (
  sensor_id INT AUTO_INCREMENT PRIMARY KEY,
  site_id INT NOT NULL,
  sensor_num INT NOT NULL,
  sensor_label VARCHAR(100),
  sensor_decimal VARCHAR(24) NOT NULL UNIQUE,
  sensor_uri VARCHAR(255),
  sensor_status ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
  date_added DATE,
  notes TEXT,
  FOREIGN KEY (site_id) REFERENCES sites(site_id) ON DELETE CASCADE
);
```

### Milesight Sensors Table
```sql
CREATE TABLE milesight_sensors (
  sensor_id INT AUTO_INCREMENT PRIMARY KEY,
  site_id INT NOT NULL,
  name VARCHAR(100),
  description TEXT,
  deveui VARCHAR(16) NOT NULL UNIQUE,
  application VARCHAR(100),
  deviceprofile VARCHAR(100),
  appkey VARCHAR(32),
  sensor_label VARCHAR(100),
  sensor_status ENUM('None', 'ACTIVE', 'INACTIVE') DEFAULT 'None',
  date_added DATE,
  notes TEXT,
  trashed BOOLEAN DEFAULT 0,
  FOREIGN KEY (site_id) REFERENCES sites(site_id) ON DELETE CASCADE
);
```

### BACnet Values Table
```sql
CREATE TABLE milesight_bacnet_values (
  bacnet_id INT AUTO_INCREMENT PRIMARY KEY,
  site_id INT NOT NULL,
  sensor_id INT NOT NULL,
  devicename VARCHAR(100),
  objectname VARCHAR(100),
  objecttype VARCHAR(50),
  unit VARCHAR(10),
  loraobjectname VARCHAR(100),
  cov VARCHAR(20),
  covincrement VARCHAR(20),
  description TEXT,
  polarity VARCHAR(50),
  inactivetext VARCHAR(100),
  activetext VARCHAR(100),
  relinquishdefault VARCHAR(100),
  FOREIGN KEY (site_id) REFERENCES sites(site_id) ON DELETE CASCADE,
  FOREIGN KEY (sensor_id) REFERENCES milesight_sensors(sensor_id) ON DELETE CASCADE
);
```

## 🎨 Customization

### Color Scheme
Modify colors in `src/constants/colors.js`:
```javascript
export const COLORS = {
  PRIMARY: '#2196F3',
  SUCCESS: '#4CAF50',
  WARNING: '#FF9800',
  ERROR: '#F44336',
  // ... more colors
};
```

### Scanner Configuration
Adjust scanner settings in `src/constants/config.js`:
```javascript
export const APP_CONFIG = {
  SCAN_COOLDOWN_MS: 10000,
  DEBOUNCE_MS: 1500,
  CAMERA_TIMEOUT_MS: 120000,
};
```

## 🔐 Security Notes

- Current authentication is mock-based for development
- Implement proper JWT-based auth in production
- Add rate limiting to API endpoints
- Use HTTPS in production
- Secure database credentials with environment variables
- Implement role-based access control (Admin/Technician)

## 📝 Usage Flow

1. **Login** → Enter credentials (currently accepts any username/password)
2. **Sites List** → View, search, or add sites
3. **Select Site** → Tap a site to manage sensors
4. **Sensor Management** → View three tabs:
   - **Inovonics**: List of Inovonics sensors
   - **Milesight**: List of Milesight sensors with apartments
   - **BACnet**: Auto-generated BACnet values
5. **Scan Sensors** → Tap "Scan" button
   - Choose mode (Inovonics/Milesight)
   - Point camera at barcode
   - Wait for success confirmation
   - 10-second cooldown before next scan
6. **Export Data** → Tap "Export CSV" on any tab to share/save data

## 🐛 Troubleshooting

### Camera not working
- Check camera permissions in device settings
- Ensure camera is not being used by another app
- Restart the app

### Network errors
- Verify API_BASE_URL is correct
- Check backend server is running
- Ensure device and server are on same network (for local dev)
- Check firewall settings

### Duplicate scans
- Wait for 10-second cooldown to complete
- Check if sensor already exists in database
- Clear app cache if needed

## 📄 License

This project is proprietary and confidential.

## 👥 Support

For issues or questions, contact the development team.
