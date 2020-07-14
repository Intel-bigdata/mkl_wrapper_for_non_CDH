# mklWrapper
## An MKL wrapper which directly bind with Intel MKL.

### Setting and Testing:

- Assume the MKL installation location is /opt/intel/mkl (you cannot just copy libmkl_rt.so).
  - Copy mkl_wrapper.jar to /opt/intel/mkl/wrapper/mkl_wrapper.jar
  - Copy mkl_wrapper.so to /opt/intel/mkl/wrapper/mkl_wrapper.so
  - By default, the MKL wrapper will search /opt/intel/mkl/wrapper to find mkl_wrapper.so. If you put it on other path, please use -Dcom.intel.mkl.wrapper=/path/to/find/mkl_wrapper.so. 
- Spark configuration. (on each node):
  - Create: /etc/ld.so.conf.d/mkl_blas.conf
  - Input: /opt/intel/mkl/lib/intel64 (your MKL library Dir) to mkl_blas.conf 
  - Run: ldconfig
- A simple sanity check to see if native MKLBLAS/LAPACK libs can be properly loaded:

   bin/spark-shell  --driver-class-path $CLASSPATH:/opt/intel/mkl/wrapper/mkl_wrapper.jar

scala> import com.github.fommil.netlib.BLAS;

scala> System.setProperty("com.github.fommil.netlib.BLAS", "com.intel.mkl.MKLBLAS")

scala> System.out.println(BLAS.getInstance().getClass().getName());

       OUTPUT: com.intel.mkl.MKLBLAS

scala> import com.github.fommil.netlib.LAPACK;

scala> System.setProperty("com.github.fommil.netlib.LAPACK", "com.intel.mkl.MKLLAPACK")

scala> System.out.println(LAPACK.getInstance().getClass().getName());

       OUTPUT: com.intel.mkl.MKLLAPACK

### How to use it: 

- It is simple to use this MKL Wrapper, just with the following option to submit your job.
  - -Dcom.github.fommil.netlib.BLAS=com.intel.mkl.MKLBLAS
  - -Dcom.github.fommil.netlib.LAPACK=com.intel.mkl.MKLLAPACK


### Example 1 (check MKL can be successfully loaded): 

- bin/spark-submit --master spark://sr576:7077
  - --class TestMKLWrapper
  - --conf "spark.executor.extraJavaOptions=-Dcom.github.fommil.netlib.BLAS=com.intel.mkl.MKLBLAS -Dcom.github.fommil.netlib.LAPACK=com.intel.mkl.MKLLAPACK"
  - --conf "spark.executor.extraClassPath=/opt/intel/mkl/wrapper/mkl_wrapper.jar"
  - --conf "spark.executorEnv.MKL_VERBOSE=1"
  - target/scala-2.11/test-netflix-mllib_2.11-1.0.jar

### Example 1 (Real test): 

- bin/spark-submit --master spark://sr576:7077
  - --class TestMKLWrapper
  - --conf "spark.executor.extraJavaOptions=-Dcom.github.fommil.netlib.BLAS=com.intel.mkl.MKLBLAS -Dcom.github.fommil.netlib.LAPACK=com.intel.mkl.MKLLAPACK"
  - --conf "spark.executor.extraClassPath=/opt/intel/mkl/wrapper/mkl_wrapper.jar"
  - target/scala-2.11/test-netflix-mllib_2.11-1.0.jar

### For some MLLIB algorithm (SVD), you also need to set driver extraJavaOptions and driver extraClassPath
