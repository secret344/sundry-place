# 矩阵乘法与分治

```java
import java.util.Date;
import java.util.Random;

public class strassen {
    static int leftSize = 1;

    /**
     * 矩阵加法
     */
    private static void Add(int[][] matrixA, int[][] matrixB, int[][] result, int size) {
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                result[i][j] = matrixA[i][j] + matrixB[i][j];
            }
        }
    }

    /**
     * 矩阵减法
     */
    private static void Sub(int[][] matrixA, int[][] matrixB, int[][] result, int size) {
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                result[i][j] = matrixA[i][j] - matrixB[i][j];
            }
        }
    }

    private static void StrassenMul(int[][] matrixA, int[][] matrixB, int[][] result, int size) {
        if (size <= leftSize) {
            // 当递归到最后一项时 size = 1, 直接返回积
            NormalMul(matrixA, matrixB, result, size);
        } else {
            // new int[half_size][half_size] 定义 1/4 的矩阵
            int half_size = size / 2;
            int[][] A11, A12, A21, A22; // 代表 matrixA 被切割成的 4 块
            int[][] B11, B12, B21, B22; // 代表 matrixB 被切割成的 4 块
            A11 = new int[half_size][half_size];
            A12 = new int[half_size][half_size];
            A21 = new int[half_size][half_size];
            A22 = new int[half_size][half_size];

            B11 = new int[half_size][half_size];
            B12 = new int[half_size][half_size];
            B21 = new int[half_size][half_size];
            B22 = new int[half_size][half_size];

            // 矩阵乘法 (AB)ij = (k = 1 -> size) Aik * Bkj
            // C11 = A11B11 + A12B21
            // C12 = A11B12 + A12B22
            // C21 = A21B11 + A22B21
            // C22 = A21B12 + A22B22
            int[][] C11, C12, C21, C22; // 代表 result 的块结果
            C11 = new int[half_size][half_size];
            C12 = new int[half_size][half_size];
            C21 = new int[half_size][half_size];
            C22 = new int[half_size][half_size];
            int[][] M1, M2, M3, M4, M5, M6, M7;
            M1 = new int[half_size][half_size];
            M2 = new int[half_size][half_size];
            M3 = new int[half_size][half_size];
            M4 = new int[half_size][half_size];
            M5 = new int[half_size][half_size];
            M6 = new int[half_size][half_size];
            M7 = new int[half_size][half_size];

            // 定义矩阵模板（算法最多用到2个额外矩阵）
            int[][] MatrixTemp1, MatrixTemp2;
            MatrixTemp1 = new int[half_size][half_size];
            MatrixTemp2 = new int[half_size][half_size];
            // 新建的矩阵赋值
            for (int i = 0; i < half_size; i++) {
                for (int j = 0; j < half_size; j++) {
                    A11[i][j] = matrixA[i][j];
                    A12[i][j] = matrixA[i][j + half_size];
                    A21[i][j] = matrixA[i + half_size][j];
                    A22[i][j] = matrixA[i + half_size][j + half_size];

                    B11[i][j] = matrixB[i][j];
                    B12[i][j] = matrixB[i][j + half_size];
                    B21[i][j] = matrixB[i + half_size][j];
                    B22[i][j] = matrixB[i + half_size][j + half_size];
                }
            }
            // 新的矩阵 作用将递归减少到 7 个
            // 第一个递归(下面同理 不在复述)
            // M1 = (A11 + A22)(B11 + B22)
            Add(A11, A22, MatrixTemp1, half_size);
            Add(B11, B22, MatrixTemp2, half_size);
            StrassenMul(MatrixTemp1, MatrixTemp2, M1, half_size);

            // M2 = (A21 + A22)B11
            Add(A21, A22, MatrixTemp1, half_size);
            StrassenMul(MatrixTemp1, B11, M2, half_size);

            // M3 = A11(B12 - B22)
            Sub(B12, B22, MatrixTemp1, half_size);
            StrassenMul(A11, MatrixTemp1, M3, half_size);

            // M4 = A22(B21 - B11)
            Sub(B21, B11, MatrixTemp1, half_size);
            StrassenMul(A22, MatrixTemp1, M4, half_size);

            // M5 = (A11 + A12)B22
            Add(A11, A12, MatrixTemp1, half_size);
            StrassenMul(MatrixTemp1, B22, M5, half_size);

            // M6 = (A21 - A11)(B11 + B12)
            Sub(A21, A11, MatrixTemp1, half_size);
            Add(B11, B12, MatrixTemp2, half_size);
            StrassenMul(MatrixTemp1, MatrixTemp2, M6, half_size);

            // M7 = (A12 - A22)(B21 + B22)
            Sub(A12, A22, MatrixTemp1, half_size);
            Add(B21, B22, MatrixTemp2, half_size);
            StrassenMul(MatrixTemp1, MatrixTemp2, M7, half_size);

            // 新矩阵计算完毕 计算C
            // C11 = M1 + M4 - M5 + M7
            Add(M1, M4, C11, half_size);
            Sub(C11, M5, C11, half_size);
            Add(C11, M7, C11, half_size);

            // C12 = M3 + M5
            Add(M3, M5, C12, half_size);

            // C21 = M2 + M4
            Add(M2, M4, C21, half_size);

            // C22 = M1 - M2 + M3 + M6
            Sub(M1, M2, C22, half_size);
            Add(C22, M3, C22, half_size);
            Add(C22, M6, C22, half_size);

            // 计算完毕 赋值给 result
            for (int i = 0; i < half_size; i++) {
                for (int j = 0; j < half_size; j++) {
                    result[i][j] = C11[i][j];
                    result[i][j + half_size] = C12[i][j];
                    result[i + half_size][j] = C21[i][j];
                    result[i + half_size][j + half_size] = C22[i][j];
                }
            }
        }
    }

    /**
     * 朴素矩阵乘法
     */
    private static void NormalMul(int[][] matrixA, int[][] matrixB, int[][] result, int size) {
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                result[i][j] = 0;
                for (int k = 0; k < size; k++) {
                    // a[i][k] * b[k][j]
                    result[i][j] += matrixA[i][k] * matrixB[k][j];
                }
            }
        }
    }

    private static int SumMatrix(int[][] matrix, int size) {
        int sum = 0;
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                sum += matrix[i][j];
            }
        }
        return sum;
    }

    /**
     * 创建size大小的(范围 0 ~ 100)矩阵
     */
    private static int[][] ProductionMatrix(int size) {
        Random r = new Random();
        int[][] result = new int[size][size];
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                result[i][j] = r.nextInt(10);
            }
        }
        return result;
    }

    public static void main(String[] args) {
        // 未作奇数补行，补列，所以size必须为 2的n次幂
        // 因为每一次都取的 n / 2
        int size = 2048;
        leftSize = 514;
        // 新建两个矩阵
        int[][] matrix1 = ProductionMatrix(size);
        int[][] matrix2 = ProductionMatrix(size);

        long startTime = new Date().getTime();
        int[][] result1 = new int[size][size];
        System.out.println("朴素算法开始时间:" + startTime);
        NormalMul(matrix1, matrix2, result1, size);
        long endTime = new Date().getTime();
        // n ^ 3
        System.out.println("朴素算法结束时间:" + endTime);
        System.out.println("朴素算法用时:" + (endTime - startTime));
        System.out.println("朴素算法和:" + SumMatrix(result1, size));

        startTime = new Date().getTime();
        int[][] result2 = new int[size][size];
        System.out.println("strassen算法开始时间:" + startTime);
        StrassenMul(matrix1, matrix2, result2, size);
        endTime = new Date().getTime();
        // n ^ lg7 ≈ n ^ 2.81
        System.out.println("strassen算法结束时间:" + endTime);
        System.out.println("strassen算法用时:" + (endTime - startTime));
        System.out.println("strassen算法和:" + SumMatrix(result2, size));

    }
}
```
