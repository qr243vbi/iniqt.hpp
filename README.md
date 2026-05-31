# inipp-qt

A lightweight, header-only, type-safe INI parser and generation library written entirely in modern C++ and optimized specifically for the **Qt framework**. 

This repository is a port of the original standard-library-dependent `inipp` library, redesigned from the ground up to eliminate dependency on standard stream libraries (`iostream`, `sstream`) and native string manipulations, replacing them with highly optimized Qt alternatives like `QString`, `QMap`, `QList`, and `QVariant`.

## 🚀 Features

*   **Header-Only:** Drop the single header into your project and you are ready to go.
*   **Pure Qt Native:** Uses `QString` natively, avoiding unnecessary conversions (`toStdString()` / `fromStdString()`).
*   **Variable Interpolation:** Supports multi-pass symbol substitutions (e.g., `url = ${protocol}://example.com`).
*   **Type-Safe Property Mapping:** Extract and assign configuration entries safely with template-deduced `get_value` and `set_value` methods backed by `QVariant` conversion systems.
*   **File Stream-Agnostic:** Parses from and serializes directly to `QTextStream`, allowing integration with `QFile`, `QTcpSocket`, `QByteArray`, and resource files (`:/`).

## 📋 Prerequisites

*   C++17 or newer (uses `if constexpr`).
*   Qt 5.15 or Qt 6.x (Core module).

## 🛠️ Usage Example

### 1. Simple Load, Modify, and Save Pipeline

```cpp
#include "inipp.hpp"
#include <QFile>
#include <QDebug>

void manageConfigurations() {
    inipp::Ini ini;

    // --- 1. Parse an Existing INI File ---
    QFile inputFile("config.ini");
    if (inputFile.open(QIODevice::ReadOnly | QIODevice::Text)) {
        QTextStream inStream(&inputFile);
        ini.parse(inStream);
        inputFile.close();
    }

    // Process comments and variable expansions
    ini.strip_trailing_comments();
    ini.interpolate();

    // --- 2. Extract Values Safely (get_value) ---
    auto &networkSec = ini.sections["Network"];
    
    QString host;
    int port = 8080; // default value if missing
    bool useProxy = false;

    if (inipp::get_value(networkSec, "Host", host)) {
        qDebug() << "Loaded Host:" << host;
    }
    inipp::get_value(networkSec, "Port", port);       // Deduce 'int' automatically
    inipp::get_value(networkSec, "UseProxy", useProxy); // Handles "true", "yes", "1"

    // --- 3. Mutate / Add Values (set_value) ---
    auto &updaterSec = ini.sections["Updater"];
    inipp::set_value(updaterSec, "CheckInterval", 7);        // Saves as "7"
    inipp::set_value(updaterSec, "AutoDownload", true);     // Saves as "true"
    inipp::set_value(updaterSec, "MirrorURL", "https://example.com");

    // --- 4. Serialize Modifications Back to Disk ---
    QFile outputFile("config_modified.ini");
    if (outputFile.open(QIODevice::WriteOnly | QIODevice::Text)) {
        QTextStream outStream(&outputFile);
        ini.generate(outStream);
        outputFile.close();
        qDebug() << "Configuration state saved successfully.";
    }
}
```

### 2. Variable Interpolation Syntax

The library natively handles nested variables out of the box using sections or local names.

```ini
[Paths]
root_dir = /home/user/app
log_dir = \${root_dir}/logs

[Network]
protocol = https
api_endpoint = \${protocol}://://example.com
```

Calling `ini.interpolate()` automatically evaluates `log_dir` down to `/home/user/app/logs`.

## ⚙️ Custom Formatting

You can customize the characters utilized for headers, assignments, comments, and variables by overriding the `Format` class structural mapping tokens:

```cpp
// Create an INI format that interprets '#' as comments and ':' for assignments
auto customFormat = QSharedPointer<inipp::Format>::create('[', ']', ':', '#', '\$', '{', ':', '}');
inipp::Ini ini(customFormat);
```

## 📄 License

Distributed under the GPL-3.0 license. See `LICENSE` for more information.
