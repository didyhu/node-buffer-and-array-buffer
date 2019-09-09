# Node Buffer, ArrayBuffer and TypedArray

这个项目是对 Node Buffer, ArrayBuffer and TypedArray 的关系说明.

1. ArrayBuffer 是 TypedArray 的数据源, TypedArray 是 ArrayBuffer 的视图. 多个 TypedArray 可以共享同一个 ArrayBuffer.

    ```js
    // 创建一个4字节的 ArrayBuffer 作为数据源
    const arrayBuffer = new ArrayBuffer(4)
    // 创建一个 Uint8Array 作为 arrayBuffer 的数据视图
    const uint8Array = new Uint8Array(arrayBuffer)
    // 创建一个 Int8Array 作为 arrayBuffer 的数据视图
    const int8Array = new Int8Array(arrayBuffer)
    // uint8Array 和 int8Array 的数据源都是同一个 arrayBuffer
    assert(uint8Array.buffer == arrayBuffer)
    assert(int8Array.buffer == arrayBuffer)
    const array = [0, 1, 2, 3]
    // 通过 uint8Array 写入数据
    uint8Array.set(array)
    // 通过 int8Array 写入数据
    int8Array[0] = 1
    int8Array[2] = -1
    // 都影响了以 arrayBuffer 为数据源的 uint8Array 和 int8Array
    assert(int8Array[0] == uint8Array[0])
    assert(int8Array[1] == uint8Array[1])
    assert(int8Array[2] == -1)
    assert(uint8Array[2] == 255)
    // 特别的 array 不会被影响
    Array.from([0, 1, 2, 3]).forEach((value, index) => {
        assert(value == array[index])
    })
    ```

2. 从 Array 拷贝数据创建 Buffer

    ```js
    // 拷贝 Array 创建 Buffer
    const array = [0, 1, 2, 3, 256, 257]
    const buffer = Buffer.from(array)
    assert(buffer[0] == 0)
    assert(buffer[1] == 1)
    assert(buffer[2] == 2)
    assert(buffer[3] == 3)
    assert(buffer[4] == 0) // 256 % 256 => 0
    assert(buffer[5] == 1) // 257 % 256 => 1
    buffer.fill(0)
    // 修改 Buffer 数据不会影响原 Array
    Array.from([0, 1, 2, 3, 256, 257]).forEach((value, index) => {
        assert(value == array[index])
    })
    ```

3. 从 String 拷贝数据创建 Buffer

    ```js
    // 拷贝 String 创建 Buffer
    const string = "hello"
    const buffer = Buffer.from(string)
    buffer.fill(0)
    // 修改 Buffer 数据不会影响原 String
    assert(string == "hello")
    ```

4. 以 ArrayBuffer 为数据源创建 Buffer

    ```js
    // 以 ArrayBuffer 为数据源创建 Buffer
    const array = [0.1, 0.2, 0.3, 0.4]
    const float32Array = new Float32Array(array)
    const arrayBuffer = float32Array.buffer
    const buffer = Buffer.from(arrayBuffer)
    // 通过 Buffer 修改 ArrayBuffer
    buffer.writeFloatLE(0.0, 0)
    // 从 Float32Array 读出
    assert(float32Array[0] == 0)
    // 通过 Float32Array 修改 ArrayBuffer
    float32Array[1] = 0.9
    // 从 Buffer 读出
    assert(buffer.readFloatLE(0) - 0.9 < Number.EPSILON)
    // Buffer 和 Float32Array 的数据源为同一个 ArrayBuffer
    assert(buffer.buffer == arrayBuffer)
    assert(buffer.buffer == float32Array.buffer)
    ```

5. 可以将 NodeBuffer 视为 Uint8Array 的一个部分的实现

    ```js
    // 可以将 NodeBuffer 视为 Uint8Array 的一个部分的实现
    const uint8Array = new Uint8Array(4)
    const buffer = Buffer.from(uint8Array.buffer)
    uint8Array.set([0, 1, 2, 3])

    // 以下字段 Buffer 和 Uint8Array 一致
    assert(uint8Array.buffer == buffer.buffer)
    assert(uint8Array.byteLength == buffer.byteLength)
    assert(uint8Array.byteOffset == buffer.byteOffset)
    assert(uint8Array.length == buffer.length)

    // 以下方法 Buffer 和 Uint8Array 功能一致
    uint8Array.fill(1, 0, 1)
    assert(buffer[0] == 1)
    buffer.fill(2, 1, 2)
    assert(uint8Array[1] == 2)
    for (let i = 0; i < 2; i++) {
        assert(buffer.slice(0, 2)[i] == uint8Array.slice(0, 2)[i])
    }

    // Uint8Array 的 sort 等方法会影响援引相同 ArrayBuffer 的 Buffer
    uint8Array.sort((a, b) => b - a)
    for (let i = 0; i < 4; i++) {
        assert(buffer[i] == uint8Array[i])
    }
    ```

6. TypedArray 和 Buffer 的相互转化

    TypedArray 和 Buffer 的数据源都是 BufferArray, TypedArray 和 Buffer 都可以作为 BufferArray 的数据视图. 

    ```js
    // typedArray to buffer
    const buffer = Buffer.from(int32Array.buffer)
    // buffer to typedArray
    const int32Array = new Int32Array(buffer.buffer)
    ```
