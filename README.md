# Matalab.engine cheatsheet for Pythoners!

Collections of FAQ about how to use python to operate Matlab.


# How to install 
two ways to install matlab.engine
* use `pip`: `pip install matlabengine`
* install specific Matlab version: `cd <MATLAB_ROOT>/extern/engines/python; python setup.py install`


# Install matlab.engine to a virtual environment?

install with `--prefix`
`python setup.py install --prefix="_your_venv_path_"`

# Start MATLAB Desktop
```python
import matlab.engine
matlab.engine.start_matlab()
matlab.engine.desktop(nargout=0)
...
matlab.engine.quit()
```
Or
```python
import matlab.engine
matlab.engine.start_matlab(option='desktop')
...
matlab.engine.quit()
```

# Start MATLAB a little faster

```python
matlab.engine.start_matlab(option="-nojvm -nosplash")
```
Note: can not open desktop by `matlab.engine.desktop` after `-nojvm` option.


# Start MATLAB -   asynchronously

```python
import matlab.engine
import time

future = matlab.engine.start_matlab(option="-desktop", background=True)

print("Starting MATLAB....")
eng = future.result()
print(eng.version())

# generating codes of modelname.slx
eng.load_system("modelName", nargout=0)
future = eng.rtwbuild("modelName", background=True)
print("generating codes...")
res = future.result()
```

# Execute commands
1 **matlab.engine.`command`()**
examples are:
```python
matlab.engine.addpath("./hdrt")
eng = matlab.engine
eng.workspace['x'] = eng.eval('-1:0.1:1')
res = eng.eval("eval('sin(0.1)')")
eng.open("modelName")
eng.set_param(model_name, attr, value, nargout=0)
```
command can be a built-in or user-defined command.

2 **getattr(matlab.engine, command)(nargout=0)**
This method can carry out a whole sentence, examples are:
```python
getattr(matlab.engine, 'eval')(line, nargout=0)
```
equivalent to
```python
matlab.engine.eval(code, nargout=0)
```
The official manual recommands to use the argument`background=True` to carry out the execution asynchronously.

# nargout
Argument `nargout` specify the number of returns, `nargout=0` supresses the returns.
```python
t = eng.gcd(100.0,80.0,nargout=3)
print(t)
```

# Redirect output
Redirect output and error
```python
import os
import time
out = io.StringIO()
err = io.StringIO()
eng.eval(codeLine, nargout=0, stdout=out, stderr=err)
print(out.getvalue())
print(err.getvalue())
```
when the code is carried out asynchronously.
```python
import os
import time
out = io.StringIO()
err = io.StringIO()
future = eng.eval(codeLine, nargout=0, stdout=out, stderr=err, background=True)
while not future.done():
	time.sleep(1)
	print(out.getvalue())
	print(err.getvalue())
```

# Handle exception

```python
try:
    future = eng.rtwbuild("b2lrc5ms", background=True, stdout=out, stderr=err)
    print("Generating codes...")
except matlab.engine.MatlabExecutionError as e:
    print("meet an error while generating codes..")
    print(e)
