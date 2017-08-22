# Interconnect Data File Parser


## Interconnect file format

According to [[1]](https://kb.lumerical.com/en/ref_s-parameter-file-formats.html) the file format follows the logic described bellow:

```
("output port name","mode label",mode ID (out),"input port name",mode ID (in),"type")
(number of frequency points, number of columns)
f1 abs(S) angle(S)
f2 abs(S) angle(S)
```

## Using the parser

```python
parsed = IParser('dircoup.dat')
```

The parser generates a S-matrix object that is an dictionary where the keys uses the format ```input_port:output_port```. So, to get the S-matrix parameters from, for instance, ```port 1``` to ```port 2``` you can do:

```Python
s_matrix = parsed_s_matrix
s_1_3 = s_matrix["port 1:port 3"]
```

The variable ```s_1_3``` holds a function that accepts the wavelenght as an argument and returns the equivalent S-parameter.

## Usign as a CapheModelView

To use it with IPKISS as a Caphe model all you have to do is:

1. Define your PCell Netlist to use the same terminology as you Lumerical Interconnect model;
2. Create an InterconnectSModelView;
3. Indicate the Lumerical Interconnect data file at simulation time;

```python
from i_to_caphe import InterconnectSModelView

# Define the Model
class DirectionalCoupler(i3.PCell):

    class Netlist(i3.NetlistView):
        # We define our terms.
        # They MUST use the same name as in your Lumerical Interconnect model
        def _generate_terms(self, terms):
            terms += i3.OpticalTerm(name="port 1")
            terms += i3.OpticalTerm(name="port 2")
            terms += i3.OpticalTerm(name="port 3")
            terms += i3.OpticalTerm(name="port 4")
            return terms


    class InterconnectSModel(InterconnectSModelView):
        data_file='dircoup.dat' # Tells IPKISS which Lumerical Interconnect data file to use


 # And run the simulation:
 my_dc = DirectionalCoupler()
 my_dc_cm = my_dc.InterconnectSModel()
 wavelengths =  np.arange(1.545e-6, 1.555e-6, 0.0005e-6)
 my_engine = i3.CapheFrequencyEngine()

 my_simulation = my_engine.SMatrixSimulation(
     model=my_dc_cm,
     wavelengths=wavelengths
 )

 my_simulation.run()

```

### Mapping terms names

If you use different names for your Lumerical Interconnect ports and your IPKISS Netlist, you can still map the names by defining the ```terms_map``` dictionary as ilustred bellow:

```python
class DirectionalCoupler(i3.PCell):

    class Netlist(i3.NetlistView):
        # We define our terms.
        def _generate_terms(self, terms):
            terms += i3.OpticalTerm(name="in1")
            terms += i3.OpticalTerm(name="in2")
            terms += i3.OpticalTerm(name="out1")
            terms += i3.OpticalTerm(name="out2")
            return terms


    class InterconnectSModel(InterconnectSModelView):

        def _default_data_file(self):
            return 'dircoup.dat'

        def _default_terms_map(self):
            """
                Map our terms names to Lumerical Interconnect port names
            """
            return {'in1:port 1',
                    'in2:port 2',
                    'out1:port 3',
                    'out2:port 4',
                    }

```
