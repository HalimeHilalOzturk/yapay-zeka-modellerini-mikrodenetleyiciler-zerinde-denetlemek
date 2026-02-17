#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>

typedef enum {
    TENSOR_FLOAT32,
    TENSOR_FLOAT16,
    TENSOR_INT8
} TensorType;

typedef union {
    float* f32;
    uint16_t* f16;
    int8_t* i8;
} TensorData;

typedef struct {
    TensorType type;
    uint32_t size;
    TensorData data;
} Tensor;

Tensor* create_tensor(uint32_t size, TensorType type) {
    Tensor* t = (Tensor*)malloc(sizeof(Tensor));
    if (!t) return NULL;
    
    t->type = type;
    t->size = size;

    size_t element_size;
    switch (type) {
        case TENSOR_FLOAT32: element_size = sizeof(float); break;
        case TENSOR_FLOAT16: element_size = sizeof(uint16_t); break;
        case TENSOR_INT8:    element_size = sizeof(int8_t); break;
        default: free(t); return NULL;
    }

    t->data.f32 = (float*)malloc(size * element_size);
    if (!t->data.f32) {
        free(t);
        return NULL;
    }
    return t;
}

void free_tensor(Tensor* t) {
    if (t) {
        free(t->data.f32);
        free(t);
    }
}

void run_example_app() {
    uint32_t n = 5;
    Tensor* f32_t = create_tensor(n, TENSOR_FLOAT32);
    Tensor* i8_t = create_tensor(n, TENSOR_INT8);

    float raw_data[] = {0.10f, 0.45f, 0.75f, 0.90f, 1.00f};
    
    for(uint32_t i = 0; i < n; i++) {
        f32_t->data.f32[i] = raw_data[i];
    }

    for(uint32_t i = 0; i < n; i++) {
        i8_t->data.i8[i] = (int8_t)(f32_t->data.f32[i] * 127);
    }

    printf("Float32: ");
    for(uint32_t i = 0; i < n; i++) printf("%.2f ", f32_t->data.f32[i]);
    
    printf("\nInt8 (Quantized): ");
    for(uint32_t i = 0; i < n; i++) printf("%d ", i8_t->data.i8[i]);
    printf("\n");

    free_tensor(f32_t);
    free_tensor(i8_t);
}

int main() {
    run_example_app();
    return 0;
}
