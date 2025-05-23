# I2CDevices – Interface générique I2C pour ESP-IDF

`I2CDevices` est une classe C++ moderne permettant de gérer simplement un périphérique I2C sous ESP-IDF. Elle offre une abstraction propre avec mutex intégré, gestion d’erreur via `std::expected`, et permet de réutiliser un bus existant ou d’en créer un nouveau.

---

## 📦 Fonctions principales

- Initialisation automatique du bus ou utilisation d’un bus partagé
- Protection thread-safe via FreeRTOS `SemaphoreHandle_t`
- API moderne en C++ avec `std::expected` pour la gestion d’erreurs
- Méthodes génériques pour écrire/lire des registres sur n’importe quel périphérique I2C

---

## 🔧 Installation

Placer les fichiers `I2CDevices.hpp` et `I2CDevices.cpp` dans le dossier `components/i2c_device/` de votre projet ESP-IDF.

Ajoutez dans `CMakeLists.txt` :
```cmake
set(EXTRA_COMPONENT_DIRS components/i2c_device)
```

---

## ✨ Utilisation

### Création avec un nouveau bus

```cpp
#include "I2CDevices.hpp"

std::shared_ptr<I2CDevices> i2c = std::make_shared<I2CDevices>(
    I2C_NUM_0,       // Port I2C
    GPIO_NUM_21,     // SDA
    GPIO_NUM_22,     // SCL
    400000,          // Fréquence (400 kHz)
    0x40             // Adresse du périphérique
);
```

### Création à partir d’un bus existant

```cpp
std::shared_ptr<I2CDevices> i2c = std::make_shared<I2CDevices>(
    existing_bus_handle, // Handle existant
    0x40                 // Adresse périphérique
);
```

### Écriture d’un registre

```cpp
uint8_t data[2] = { 0x12, 0x34 };
auto result = i2c->write(0x05, data, 2);

if (!result) {
    ESP_LOGE("I2C", "Erreur d'écriture : %s", result.error().what());
}
```

### Lecture d’un registre

```cpp
uint8_t buffer[2];
auto result = i2c->read(0x05, buffer, 2);

if (!result) {
    ESP_LOGE("I2C", "Erreur de lecture : %s", result.error().what());
} else {
    uint16_t value = (buffer[0] << 8) | buffer[1];
    ESP_LOGI("I2C", "Valeur lue : 0x%04X", value);
}
```

---

## 🧪 Exemple d’intégration avec un driver

```cpp
class MySensor {
public:
    MySensor(std::shared_ptr<I2CDevices> dev) : i2c(std::move(dev)) {}

    std::optional<uint16_t> readStatus() {
        uint8_t buf[2];
        auto res = i2c->read(0x01, buf, 2);
        if (!res) return std::nullopt;
        return (buf[0] << 8) | buf[1];
    }

private:
    std::shared_ptr<I2CDevices> i2c;
};
```

---

## 📄 Licence

Ce projet est distribué sous la licence **Apache License 2.0**.  
Vous pouvez l’utiliser librement dans des projets personnels ou commerciaux.

Voir le fichier [`LICENSE`](./LICENSE) pour plus d’informations.

---

## 👨‍💻 Auteur

Développé par Lionel Orcil pour IOBEWI.