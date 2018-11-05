# CHECK - DRAFT - CHECK TIMINGS - MAY BE INACCURATE

## Check threading when using Python with NumPy/SciPy

Note that pure Python code is subject to the GIL (Global Interpreter Lock). However, compiled code extensions may use threads. While most versions of NumPy do not use threads natively, the BLAS/LAPACK libraries that NumPy uses may. 

Get basic configuration including BLAS/LAPACK libraries:

    import numpy
    numpy.__config__.show()

### Interactive method

An indication of thread usage can be obtained interactively using top (or more graphical alternatives such as htop or nmon). Tip: If using top, toggle 1 to see the CPU breakdown at the top. These are logical CPUs, which may include hyperthreads. To see the breakdown of physical cores and hyperthreads look at /proc/cpuinfo. The "core id" gives the physical core ID which enables you to determine which logical CPUs share a physical core (normally thread IDs are block distributed to cores).

### Run-time analysis
    
Place this profiler in a file called prof_threads.py

```python
from time import time
import threading
import sys
from collections import deque
try:
    from resource import getrusage, RUSAGE_SELF
except ImportError:
    RUSAGE_SELF = 0
    def getrusage(who=0):
        return [0.0, 0.0] # on non-UNIX platforms cpu_time always 0.0

p_stats = None
p_start_time = None

def profiler(frame, event, arg):
    if event not in ('call','return'): return profiler
    #### gather stats ####
    rusage = getrusage(RUSAGE_SELF)
    t_cpu = rusage[0] + rusage[1] # user time + system time
    code = frame.f_code 
    fun = (code.co_name, code.co_filename, code.co_firstlineno)
    #### get stack with functions entry stats ####
    ct = threading.currentThread()
    try:
        p_stack = ct.p_stack
    except AttributeError:
        ct.p_stack = deque()
        p_stack = ct.p_stack
    #### handle call and return ####
    if event == 'call':
        p_stack.append((time(), t_cpu, fun))
    elif event == 'return':
        try:
            t,t_cpu_prev,f = p_stack.pop()
            assert f == fun
        except IndexError: # TODO investigate
            t,t_cpu_prev,f = p_start_time, 0.0, None
        call_cnt, t_sum, t_cpu_sum = p_stats.get(fun, (0, 0.0, 0.0))
        p_stats[fun] = (call_cnt+1, t_sum+time()-t, t_cpu_sum+t_cpu-t_cpu_prev)
    return profiler


def profile_on():
    global p_stats, p_start_time
    p_stats = {}
    p_start_time = time()
    threading.setprofile(profiler)
    sys.setprofile(profiler)


def profile_off():
    threading.setprofile(None)
    sys.setprofile(None)

def get_profile_stats():
    """
    returns dict[function_tuple] -> stats_tuple
    where
      function_tuple = (function_name, filename, lineno)
      stats_tuple = (call_cnt, real_time, cpu_time)
    """
    return p_stats
```

*Source for thread profiler script: (http://code.activestate.com/recipes/465831/) by Maciej Obarski*

Now in your code you can use the profiler as in the following example, which computes a Singular Value Decomposition with NumPy. 

```python
# Singular Value Decomposition (SVD)

from __future__ import print_function
import numpy as np
from time import time
import prof_threads as pt

np.random.seed(0)
size=4096
matA = np.random.random((int(size), int(size / 2)))

t = time()
pt.profile_on()
np.linalg.svd(matA, full_matrices = False)
pt.profile_off()
delta = time() - t

print("SVD of a %dx%d matrix in %0.2f s." % (size , size / 2, delta))
from pprint import pprint
pprint(pt.get_profile_stats())
```

In the output look for 'svd' to see a breakdown of time on different threads for the SVD function itself. This should give the time spent in each thread along with the location of the Python routine (in this case in NumPy) under which the threaded work occurs.

