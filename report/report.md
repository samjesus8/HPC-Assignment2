# Dynamic Intel, wants code!
## Assignment Details

- Author: ```Samuel Jesuthas```
- Module: ```MCOMD3HPC – High Performance Computing```
- Deadline: ```10th January 2024```

## Introduction

- This report will feature detailed analysis of the importance of threading in high performance applications. When a program runs multi-threaded, you are spreading out the workload to multiple workhorses, which increases the overall performance of your program.

- The program we will look at, simply generates 2, 1000x1000 matrixes with random values, and multiplies them together. It then takes the result of this and multiplies it by another 1000x1000 randomly generated matrix, and then it will do this for a 3rd time to get 3 iterations.

- Throughout this report, we will analyze the importance of threading in this application as it significantly increases the performance when the multiplications are performed. We will also look at thread pool implementations for this program and we will test how stable and scalable my solution is for business use.

## Testing Information

- Note that all the testing and program execution examples in this report, have been performed on a desktop machine with the following specs:
    - CPU: **AMD Ryzen 5 5600G**
    - RAM: **32 GB DDR4**
    - GPU: **AMD Radeon RX 6700 (10 GB GDDR6)**

- Keep in mind that the actual performance may vary based on the system and the workload.

## Task 1 - Simple Implementation (Non-Threaded)

### matrixEngine.java (Matrix Calculation Handler)

- To perform a 1000x1000 matrix multiplication, firstly, we need to generate the 2 matrixes to get our first iteration. To simplify the process, I made a class called `matrixEngine` that will handle everything for us.

- The first method `GenerateBaseMatrixes()` will generate our 2 1000x1000 matrixes which we will multiply together.

    ```java
    public matrixResult GenerateBaseMatrixes()
    {
        long[][] matrix1 = new long[1000][1000];
        long[][] matrix2 = new long[1000][1000];

        fillMatrix(matrix1);
        fillMatrix(matrix2);

        return new matrixResult(matrix1, matrix2);
    }
    ```

    - The reason I am using `long` as the data type for the matrix, is so that I can handle overflow of values. `long` can support values from `-9,223,372,036,854,775,808` up to `9,223,372,036,854,775,807`.

    - Because Java doesn't allow methods to return more than 1 data type, I made another class called `matrixResult` which stores the 2 matrixes in one object, also known as a tuple.

- The `fillMatrix()` method does a `for` loop through each value of the matrix and fills it with a value that is randomly generated from 1-100.

    ```java
    public void fillMatrix(long[][] matrix)
    {
        Random random = new Random();

        for (int i = 0; i < matrix.length; i++) {
            for (int j = 0; j < matrix[i].length; j++) {
                matrix[i][j] = random.nextInt(100);
            }
        }
    }
    ```

- The final method in this class is the `multiplyMatrices()` method, which is what we will be using to multiply 2 matrixes together.

    ```java
    public long[][] multiplyMatrices(long[][] matrix1, long[][] matrix2) 
    {
        long[][] resultMatrix = new long[1000][1000];
        for (int i = 0; i < matrix1.length; i++) {
            for (int j = 0; j < matrix2[0].length; j++) {
                for (int k = 0; k < matrix2.length; k++) {
                    resultMatrix[i][j] += matrix1[i][k] * matrix2[k][j];
                }
            }
        }

        return resultMatrix;
    }
    ```

    - The triple-nested `for` loop in this method is what performs the multiplication. What it does is iterate through each element of the 2 matrixes and perform dot-product matrix multiplication
        - `i` will iterate over every ROW of `matrix1`
        - `j` will iterate over every COLUMN of `matrix2`
        - `k` will iterate over the COLUMNS of `matrix1` and the ROWS of `matrix2`

- As we are iterating over every element of the matrix, we get the product of the current row of `matrix1` and then multiply it by the product of the current column of `matrix2`. We assign the results of this to the corresponding element of the `resultMatrix` matrix

    ```java
    resultMatrix[i][j] += matrix1[i][k] * matrix2[k][j];
    ```

### App.java (The main program)

- The main application will run based on what option you select. You have the option to run the matrix program in a threaded or non-threaded manner

    ```
    Please choose an option to run this program 
    1. Run /w No Threading
    2. Run /w Threading
    3. Run using Thread Pool
    4. Verify Threaded vs Non-Threaded
    5. Exit Program
    ```

- The non-threaded method takes in 2 parameters `matrix1` & `matrix2` which are the starting 2 matrixes we need to perform the 1st multiplication. We are also returning a `long[][]` which is the result matrix, in order to compare the result against the threaded versions

    ```java
    private static long[][] performMatrixMultiplicationNoThreading(long[][] matrix1, long[][] matrix2)
    {

    }
    ```

- Next, we are displaying the 2 starting matrixes by using the `printMatrixPreview()` method. This method allows us to view a variable length portion of the matrix, in this case the first 10x10.

    ```java
    //Initialize MatrixEngine
    var MatrixEngine = new matrixEngine();

    //Display the 2 starting matrixes which will be multiplied together to give our first iteration
    System.out.println("Matrix 1: \n");
    printMatrixPreview(matrix1, 10, 10); //We will only display the first 10x10 portion of the matrix
    System.out.println("Matrix 2: \n");
    printMatrixPreview(matrix2, 10, 10);
    ```

- Now we will perform our first multiplication by using the `multiplyMatrices()` method.

    ```java
    //Executing the multiplication method to get our first iteration
    var result1 = MatrixEngine.multiplyMatrices(matrixes.matrix1, matrixes.matrix2);
    System.out.println("1st Multiplication /w No Threading: \n");
    printMatrixPreview(result1, 10, 10);
    ```

    - Since the method returns a `long[][]` it would be best to store this in a variable, which is what `result1` is.

- Now we have our first iteration, all we have to do is multiply 2 more times. We take the output of `result1` and multiply it by a brand new 1000x1000 matrix

    ```java
    //Now a 2nd randomly generated 1000x1000 matrix will be created
    long[][] secondIterationMatrix = new long[1000][1000];
    MatrixEngine.fillMatrix(secondIterationMatrix);

    //We will multiply this matrix by the result of the 1st multiplication
    var result2 = MatrixEngine.multiplyMatrices(result1, secondIterationMatrix);
    System.out.println("2nd Multiplication /w No Threading: \n");
    printMatrixPreview(result2, 10, 10);
    
    //And we will do this a 3rd time, for our 3rd iteration
    long[][] thirdIterationMatrix = new long[1000][1000];
    MatrixEngine.fillMatrix(thirdIterationMatrix);

    //Multiplying the 3rd matrix by the result of the 2nd multiplication
    var result3 = MatrixEngine.multiplyMatrices(result2, thirdIterationMatrix);
    System.out.println("3rd Multiplication /w No Threading: \n");
    printMatrixPreview(result3, 10, 10);
    ```

    - `result1`, `result2` & `result3` should show an exponential increase in the output

### Output

- When we select the non-threaded option and run our program. This is what 1 run of the program will give us. Note that, each run will be different as we are randomly generating each matrix

```
Please choose an option to run this program 
1. Run with No Threading
2. Run with Threading
3. Exit Program
1
Matrix 1: 

77 72 77 37 20 70 89 76 97 26 
11 20 96 9 34 69 18 19 11 67  
46 4 35 75 37 72 18 7 57 46   
0 4 57 28 75 32 71 19 68 0    
59 21 47 31 72 97 93 94 62 28 
86 35 53 69 53 23 65 56 85 71 
24 21 55 82 64 1 27 52 34 70  
57 1 45 81 4 72 61 52 12 91 
67 6 92 99 62 92 56 36 17 36
78 65 80 84 96 47 86 50 21 26

Matrix 2:

97 18 77 20 2 89 0 87 55 96
56 45 6 13 50 81 23 62 48 18
86 47 50 96 10 65 84 84 54 5
2 44 61 3 78 23 99 35 10 13
72 76 88 97 64 82 22 74 52 83
10 69 34 63 55 64 2 88 24 77
55 36 96 35 81 80 97 17 48 99
36 99 22 12 6 30 75 23 43 22
79 23 73 95 48 9 55 78 42 95
18 87 8 91 22 32 72 28 94 48

1st Multiplication /w No Threading: 
...

2nd Multiplication /w No Threading: 
...

3rd Multiplication /w No Threading: 
...  

Execution time: 10129 milliseconds
```

- The non-threaded method takes so long, in this particular run, it took us 10,129 ms to execute the whole method.

## Task 2 - Multi-threaded Implementation

- We will now look at a threaded implementation of this program. The multiplication process will now be done by multiple threads, which should significantly increase the performance of the application

### matrixEngineThreaded.java (The thread code)

```java
public class matrixEngineThreaded extends Thread {
    private long[][] matrix1;
    private long[][] matrix2;
    private long[][] resultMatrix;
    private int startRow;
    private int endRow;
}
```

- To ensure we are utilizing threads, we can use the `extends` attribute to make this whole class run in a threaded manner

- In this class, we have some basic properties, and a constructor which allows us to pass this information in.
    - `matrix1` & `matrix2` are the matrixes passed in for multiplication
    - `resultMatrix` is used to store the result
    - `startRow` & `endRow` are used for the multiplication, as the workload has been distributed among threads. We are working with portions of a 1000x1000 matrix

- In this class, we have one main method which is the `multiply()` method, which is actually just the same code as the `multiplyMatrices()` method in the `matrixEngine` class. However, we run this method in the `run()` method, which is responsible for starting the thread

    ```java
    @Override
    public void run() {
        multiply();
    }
    ```

### matrixEngine.java (Running the thread)

- Now we will head back to the `matrixEngine` class to perform our matrix multiplication. Since most of our main logic is in that class, it would be wise to make our threaded implementation in there, to reduce unnecessary code.

- Our threaded multiplication takes place in the `multiplyMatricesThreaded()` method, which returns a `long[][]` which is the resulting matrix after the multiplication

- We start by creating an empty 1000x1000 matrix to use as the return object

    ```java
    long[][] resultMatrix = new long[1000][1000];
    ```

- Now we start creating the threads and we will calculate how many rows each thread will handle by dividing the total number of rows by the specified number of threads, which in this case is `numThreads`

    ```java
    // Calculating the distribution of workflow for these threads
    for (int i = 0; i < numThreads; i++) {
        int startRow = i * rowsPerThread;
        int endRow = (i == numThreads - 1) ? 1000 : (i + 1) * rowsPerThread;

        threads[i] = new matrixEngineThreaded(matrix1, matrix2, resultMatrix, startRow, endRow);
        threads[i].start();
    }
    ```

    - What we are doing here is `for` looping through each thread in the `threads` array

    - We are then assigning a portion of the matrix for the thread to process, which is why we have the constructor for `matrixEngineThreaded`, because we can assign it to that class, and begin the thread

    - Starting the thread, will execute the `multiply()` method

- Now the threads will begin execution and start multiplying portions of each matrix until the entirety of `matrix1` and `matrix2` are multiplied. We have to make some logic to check if the threads have finished doing their job

    ```java
    // Wait for all threads to finish
    try {
        for (int i = 0; i < numThreads; i++) {
            threads[i].join();
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    ```

    - A useful tool in java is the `try/catch` block, which ensures the application doesn't crash whenever it encounters an exception

    - We can use the `join()` method to ensure that the main thread waits for each worker thread to complete before moving on.

- Overall, this implementation divides the matrix multiplication task among multiple threads, allowing for concurrent execution and potentially speeding up the process compared to the single-threaded approach.

### App.java (Execution)

- A lot of the logic to execute this, is very similar to the `performMatrixMultiplicationNoThreading()` method. However this time, we need to use the `multiplyMatricesThreaded()` method, to run the program in a threaded manner

- This is what happens when we execute the program, with threaded support

    ```
    Please choose an option to run this program
    1. Run /w No Threading
    2. Run /w Threading
    3. Run using Thread Pool
    4. Exit Program
    2
    Matrix 1: 

    77 69 54 35 84 24 9 63 47 72  
    94 31 68 74 84 31 62 52 43 70 
    12 76 58 73 87 25 17 18 51 62 
    79 43 11 61 28 61 27 89 76 84 
    30 53 76 39 81 2 58 11 33 98  
    2 55 46 97 70 71 91 90 47 33  
    29 23 56 29 85 37 95 45 89 79 
    13 44 71 94 94 32 47 39 51 82 
    80 81 48 99 86 41 1 19 17 95  
    23 9 18 99 66 8 3 60 74 91    

    Matrix 2: 

    25 27 13 20 63 29 33 48 79 73 
    63 70 36 10 99 4 8 16 61 77   
    72 66 66 69 60 97 3 77 52 77  
    36 82 32 97 92 23 54 75 3 13  
    96 62 73 16 62 42 38 94 55 11 
    82 45 3 2 91 10 43 74 34 81   
    13 69 83 42 0 15 45 59 59 70  
    88 76 79 15 16 35 17 19 45 40 
    73 66 44 20 88 63 59 79 91 27 
    72 37 71 9 10 89 86 45 74 48

    1st Multiplication /w Threading: 
    ...

    2nd Multiplication /w Threading: 
    ...

    3rd Multiplication /w Threading:
    ...  


    Program Execution time: 1626 milliseconds
    ```

    - This is already significantly faster, only taking 1626 ms in this particular run. However, to ensure we are getting the correct result, we can compare it against the non-threaded method to see if we are getting the same result

### Comparing with the Gold Standard

- To ensure that our threaded implementation works, we need to compare the output of this, versus our non-threaded code
- Both outputs should be the same, the threaded code should only just run faster
- To check this, we can simply subtract the outputs of the threaded/non-threaded code, and the result should just be 0
- This is what the `verifyResult()` method does:

    ```java
    private static void verifyResult(long[][] result, long[][] goldStandardResult) {
        boolean verificationPassed = true;
        for (int i = 0; i < 1000; i++) {
            for (int j = 0; j < 1000; j++) {
                if (result[i][j] != goldStandardResult[i][j]) {
                    verificationPassed = false;
                    break;
                }
            }
            if (!verificationPassed) {
                break;
            }
        }

        if (verificationPassed) {
            System.out.println("Verification successful! Results match the gold standard.");
        } else {
            System.err.println("Verification failed! Results differ.");
        }
    }
    ```

    - We can `for` loop through both results and perform a simple value check to see if each value of the matrix from both matrices, is equal to each other. If there is one unequal value, we know the matrixes aren't the same and something has gone wrong

## Task 3 - Thread Pool Implementation

## Task 4 - Testing the Thread Pool