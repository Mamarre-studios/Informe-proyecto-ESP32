**inserte codigo**

```#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"
#include "esp_timer.h"

// Pines de los sensores y actuadores
#define SENSOR_IR_LEFT GPIO_NUM_4
#define SENSOR_IR_RIGHT GPIO_NUM_5
#define TRIG_PIN GPIO_NUM_18
#define ECHO_PIN GPIO_NUM_19
#define MOTOR_LEFT GPIO_NUM_21
#define MOTOR_RIGHT GPIO_NUM_22

// Umbrales
#define OBSTACLE_DISTANCE_CM 20 // Distancia mínima para detenerse

// Función para medir la distancia del sensor ultrasónico
float measure_distance() {
    gpio_set_level(TRIG_PIN, 1);
    esp_rom_delay_us(10);
    gpio_set_level(TRIG_PIN, 0);

 // Medir la duración del pulso ECHO
    while (gpio_get_level(ECHO_PIN) == 0); 
    int64_t start_time = esp_timer_get_time();
    while (gpio_get_level(ECHO_PIN) == 1); 
    int64_t end_time = esp_timer_get_time();

// Calcular distancia en cm
    float duration = (end_time - start_time) / 1000.0; 
    float distance = (duration / 2.0) * 0.0343;        

 return distance;
}

// Configuración inicial
void setup() {
    // Configuración de pines
    gpio_set_direction(SENSOR_IR_LEFT, GPIO_MODE_INPUT);
    gpio_set_direction(SENSOR_IR_RIGHT, GPIO_MODE_INPUT);
    gpio_set_direction(TRIG_PIN, GPIO_MODE_OUTPUT);
    gpio_set_direction(ECHO_PIN, GPIO_MODE_INPUT);
    gpio_set_direction(MOTOR_LEFT, GPIO_MODE_OUTPUT);
    gpio_set_direction(MOTOR_RIGHT, GPIO_MODE_OUTPUT);

// Inicializar motores apagados
    gpio_set_level(MOTOR_LEFT, 0);
    gpio_set_level(MOTOR_RIGHT, 0);
}

// Bucle principal
void loop() {
    while (1) {
        // Medir distancia del sensor ultrasónico
        float distance = measure_distance();

 if (distance < OBSTACLE_DISTANCE_CM) {
            // Obstáculo detectado, detener motores
            gpio_set_level(MOTOR_LEFT, 0);
            gpio_set_level(MOTOR_RIGHT, 0);
        } else {
            // Leer sensores IR
            int ir_left = gpio_get_level(SENSOR_IR_LEFT);
            int ir_right = gpio_get_level(SENSOR_IR_RIGHT);

 // Controlar motores según los sensores IR
            gpio_set_level(MOTOR_LEFT, ir_left);
            gpio_set_level(MOTOR_RIGHT, ir_right);
        }

// Pequeña demora para estabilidad
        vTaskDelay(pdMS_TO_TICKS(50));
    }
}

void app_main() {
    setup();
    loop();
}
