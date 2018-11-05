# Get numpy build configuration.

NumPy is based on compiled C extensions and in turn is built upon BLAS/LAPACK libraries. The following gives some information on how to interogate the build configuration and dependent libraries upon which NumPy is built.

Get basic configuration including BLAS/LAPACK libraries:

    import numpy
    numpy.__config__.show()

    
<!-- Hang on - not sure about this - does not seem to show OpenBLAS. Also may need to use readlink -e to follow links -->    
Note that configuration information may not be 100% reliable. Dependencies can also be checked by using the ldd command on numpy shared libraries. E.g.

    ldd /<path_to_site-packages>/numpy/core/multiarray.so
    
This should show if you are linked to mkl/openblas etc...    
    
Note the libraries may have variations in name based on platform (e.g. multiarray.cpython-36m-x86_64-linux-gnu.so). To check the site packages directory location you can do:

    $ python
    Python 3.6.3 |Intel Corporation| (default, Oct 16 2017, 15:28:36) ....
    >>> import numpy
    >>> numpy.__path__
    ['/home/shudson/miniconda3/lib/python3.6/site-packages/numpy']
    
    
To get full NumPy configuration details including compiler options/environment:

    import numpy.distutils
    np_config_vars = numpy.distutils.unixccompiler.sysconfig.get_config_vars()
    
    import pprint
    pprint.pprint(np_config_vars)

Or to write output to a file

    f = open('np_config_pprint.log','w')
    pprint.pprint(np_config_vars,f)
    
<!-- See below for better way
This will give a complete list of configuration variables. As these are held in a dictionary, individual items can be accessed as in the following example:

    pprint.pprint(np_config_vars['CFLAGS']) -->
    
To get specific variables you can specify them as arguments E.g:

<!--    numpy.distutils.unixccompiler.sysconfig.get_config_vars('CC', 'CXX', 'OPT', 'BASECFLAGS','CCSHARED', 'LDSHARED', 'SO') -->
    
    import numpy.distutils
    c_cpp_compiler = numpy.distutils.unixccompiler.sysconfig.get_config_vars('CC', 'CXX')
    compile_vars = numpy.distutils.unixccompiler.sysconfig.get_config_vars('CFLAGS', 'LDSHARED')
    print(c_cpp_compiler)   
    print(compile_vars)
    
If numpy has been installed without modification it will likely inherit the configuration from Python. You can check that with distutils.

<!-- needs checking -->

    from distutils import sysconfig
    sysconfig.get_config_vars()

Or with args:
      
<!--      sysconfig.get_config_vars('CC', 'CXX', 'OPT', 'BASECFLAGS',
                                'CCSHARED', 'LDSHARED', 'SO') -->

                                
      sysconfig.get_config_vars('CC', 'CXX', 'CFLAGS', 'LDSHARED')

<!-- OK - if I do above without options get same as numpy one above - even though this does not even reference numpy
     also - on interactive pythnon if just call without using variable prints nicely without needing pprint
     So it looks like this is just getting sysconfig???? - but maybe thats not always the case - perhaps if you
     have modified numpy???-->
     

Configuration details for using NumPy with MKL, along with how to build with different compiler flags can be seen in this Intel article [numpyscipy-with-intel-mkl](https://software.intel.com/en-us/articles/numpyscipy-with-intel-mkl). Note that MKL can be built with Intel or GNu compilers. Instructions exist for both, along with setting OpenMP threading support.

<!-- Add something on finding number of threads it will use -->

     
