.. _PySCeS-Inputfile:

The PySCeS Model Description Language
=====================================

PySCeS: the **\Py**\ thon **\ S**\ imulator for **\ Ce**\ llular 
**\ S**\ ystems is an extendable toolkit for the analysis and investigation of 
cellular systems. It is available for download from: http://pysces.github.io. 
This section deals with the PySCeS Model Description Language (MDL), a 
human-readable format for defining kinetic models.

PySCeS uses an ASCII text based *input file* to describe a 
cellular system in terms of its stoichiometry, kinetics, 
compartments and parameters. Input files may have any filename 
with the single restriction that, for cross platform 
compatibility, they must end with the extension *.psc*. Since version 0.7, 
the PySCeS MDL has been updated and extended to be compatible with models 
defined in the `SBML`_ standard.

We hope that you will enjoy using our software. If, however, you find any
unexpected features (i.e. bugs) or have any suggestions on how we can improve
PySCeS please let us know by opening an issue on `Github 
<https://github.com/PySCeS/pysces/issues>`_.
 
.. _PySCeS-Inputfile-Detailed:

Defining a PySCeS model
-----------------------

.. _PySCeS-Inputfile-Basic:

A kinetic model
~~~~~~~~~~~~~~~

The basic description of a kinetic model in the PySCeS MDL contains
the following information:

* whether any fixed (boundary) species are present
* the reaction network stoichiometry
* rate equations for each reaction step
* parameter and boundary species initial values
* the initial values of the variable species

Although it is in principle possible to define an ODE based model
without reactions or free species, for practical purposes PySCeS
requires a minimum of a single reaction. Once this information is
obtained it can be organised and written as a PySCeS input file.
While this list is the minimum information required for a PySCeS input
file, the MDL allows the definition of advanced models that contain
compartments, global units, functions, rate and assignment rules.

.. _PySCeS-Inputfile-Detailed-Keywords:

Model keywords
~~~~~~~~~~~~~~

Since PySCeS 0.7 it is possible to define keywords that
specify model information. Keywords have the general form

``<keyword>: <value>``

The *Modelname* (optional) keyword, containing only 
alphanumeric characters (or _), describes the model filename 
(typically used when the model is exported via the PySCeS 
interface module) while the *Description* keyword is a (short) 
single line model description. :: 

  Modelname: rohwer_sucrose1
  Description: Sucrose metabolism in sugar cane (Johann M. Rohwer)
 
Two keywords (optional) are available for use with models that have one or
more compartments defined. Both take a boolean (``True``/``False``) as
their value: 

 * *Species_In_Conc* specifies whether the species symbols used in 
   the rate equations represent a concentration (``True``, default) 
   or an amount (``False``).
 * *Output_In_Conc* tells PySCeS to output the results of numerical 
   operations on species in concentrations (``True``, default) or 
   in amounts (``False``). 
   
::
 
  Species_In_Conc: True
  Output_In_Conc: False
 

.. _PySCeS-Inputfile-Detailed-Units:

Global unit definition
~~~~~~~~~~~~~~~~~~~~~~

PySCeS 0.7+ supports the (optional) definition of a set of 
global units. In doing so we have chosen to follow the general 
approach used in the Systems Biology Modelling Language (`SBML`_
L2V3) specification. The general definition of a PySCeS unit 
is: ```<UnitType>: <kind>, <multiplier>, <scale>, <exponent>``` 
where *kind* is a string describing the base unit (for SBML 
compatibility this should be an SI unit) e.g. mole, litre, 
second or metre. The base unit is modified by the multiplier, 
scale and index using the following relationship: 
*<multiplier> * (<kind> * 10**<scale>)**<index>*. The 
default unit definitions are:  :: 

  UnitSubstance: mole, 1, 0, 1
  UnitVolume: litre, 1, 0, 1
  UnitTime: second, 1, 0, 1
  UnitLength: metre, 1, 0, 1
  UnitArea: metre, 1, 0, 2

Please note that defining these values does not affect the 
numerical analysis of the model in any way. 

.. _PySCeS-Inputfile-Detailed-Names:

Symbol names and comments
~~~~~~~~~~~~~~~~~~~~~~~~~

Symbol names (i.e. reaction, species, compartment, function, 
rule and parameter names etc.) must start with either an 
underscore or letter and be followed by any combination of 
alpha-numeric characters or an underscore. Like all other 
elements of the input file names are case sensitive:  :: 

  R1
  _subA
  par1b
  ext_1

Explicit access to the "current" time in a time simulation is 
provided by the special symbol ``_TIME_``. This is useful in 
the definition of events and rules (see section on `Advanced 
model construction`_ for more details). 

Comments can be placed anywhere in the input file in one of two 
ways, as single line comment starting with a *#* or as a Python-styled
multi-line triple quoted *"""<comment>"""*:

.. code-block:: python

  # everything after this is ignored

  """
  This is a comment
  spread over a
  few lines.
  """

.. _PySCeS-Inputfile-Detailed-Compartments:

Compartment definition
~~~~~~~~~~~~~~~~~~~~~~

By default (as is the case in all PySCeS versions < 0.7) PySCeS 
assumes that the model exists in a single unit volume 
compartment. In this case it is **not** necessary to define a 
compartment and the ODEs therefore describe changes in 
concentration per time. However, **if a compartment is defined**, 
PySCeS assumes that the ODEs describe changes in substance **amount per 
time** in keeping with the SBML standard. Doing this affects 
how the model is defined in the input 
file (especially with respect to the definitions of rate 
equations and species) and the user is **strongly** advised to 
read the User Guide before building models in this way. The 
general compartment definition is ``Compartment: <name>, 
<size>, <dimensions>``, where *<name>* is the unique 
compartment id, *<size>* is the size of the compartment (i.e. 
length, volume or area) defined by the number of *<dimensions>* 
(e.g. 1,2,3):  :: 

 Compartment: Cell, 2.0, 3
 Compartment: Memb, 1.0, 2 

.. _PySCeS-Inputfile-Detailed-Functions:

Function definitions
~~~~~~~~~~~~~~~~~~~~

A relatively recent addition to the PySCeS MDL is the ability to define 
SBML-styled functions. Simply put these are code substitutions that 
can be used in rate equation definitions to, for example, 
simplify the kinetic law. The general syntax for a function is 
``Function: <name>, <arglist> {<formula>}`` where *<name>* is the 
unique function id, *<arglist>* is one or more comma separated 
function arguments. The *<formula>* field, enclosed in curly 
braces, may only make use of arguments listed in the 
*<arglist>* and therefore **cannot** reference model attributes 
directly. If this functionality is required a forcing function/assignment rule
(see :ref:`PySCeS-Inputfile-Advanced-Assignment`) may be what you are looking 
for. :: 

 Function: rmm_num, Vf, s, p, Keq {
 Vf*(s - p/Keq)
 }

 Function: rmm_den, s, p, Ks, Kp {
 s + Ks*(1.0 + p/Kp)
 }

The syntax for function definitions has been adapted from 
`Antimony <https://tellurium.readthedocs.io/en/latest/antimony.html>`_. 

.. _PySCeS-Inputfile-Detailed-Fixed:

Defining fixed species
~~~~~~~~~~~~~~~~~~~~~~

Boundary species, also known as fixed or external species, are 
a special class of parameter used when modelling biological 
systems. The PySCeS MDL fixed species are declared on a single 
line as ``FIX: <fixedlist>``. The *<fixedlist>* is a space-separated list of 
symbol names which should be initialised like 
any other species or parameter: ::

  FIX: Fru_ex Glc_ex ATP ADP UDP phos glycolysis Suc_vac

If no fixed species are present in the model then this 
declaration should be omitted entirely. 

.. _PySCeS-Inputfile-Detailed-Reactions:

Reaction stoichiometry and rate equations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The reaction stoichiometry and rate equation are defined together
as a single reaction step. Each step in the system is defined as
having a name (identifier), a stoichiometry (substrates are
converted to products) and rate equation (the catalytic activity,
described as a function of species and parameters). All reaction
definitions should be separated by an empty line. The general format
of a reaction in a model with no compartments is: ::

  <name>: 
      <stoichiometry>
      <rate equation>

The *<name>* argument follows the syntax as discussed in 
:ref:`PySCeS-Inputfile-Detailed-Names` above;
however, when more than one compartment has 
been defined it is important to locate the reaction in its 
specific compartment. This is done using the ``@`` operator: :: 

  <name>@<compartment>: 
      <stoichiometry>
      <rate equation>

where *<compartment>* is a valid compartment name. In either 
case this then followed (either directly or on the next line) 
by the reaction stoichiometry.  

Each *<stoichiometry>* argument is defined in terms of reaction 
substrates, appearing on the left hand side, and products on the 
right hand side of an identifier which labels the reaction as 
either reversible (*=*) or irreversible (*>*). If required each 
reagent's stoichiometric coefficient (PySCeS accepts both 
integer and floating point) should be included in curly braces 
*{}* immediately preceding the reagent name. If these are 
omitted a coefficient of one is assumed.

.. code-block:: bash

  {2.0}Hex_P = Suc6P + UDP  # reversible reaction
  Fru_ex > Fru              # irreversible reaction
  species_5 > $pool         # a reaction to a sink

The PySCeS MDL also allows the use of the ``$pool`` token that 
represents a placeholder reagent for reactions that have no 
net substrate or product. The reversibility of a reaction is only 
used when exporting the model to other formats (such as SBML) 
and in the calculation of elementary modes. It does not affect 
the numerical evaluation of the rate equations in any way. 

Central to any reaction definition is the *<rate equation>* 
(SBML kinetic law). This should be written as valid Python 
expression and may fall across more than one line. Standard 
Python operators ``+ - * / **`` are supported (note the Python 
power e.g. *2^4* is written as *2\*\*4*). There is no shorthand 
for multiplication with a bracket so *-2(a+b)^h* would be written as 
*-2\*(a+b)\*\*h* and normal operator precedence applies: 

 +--------+-------------------------+
 |  +, -  | addition, subtraction   |
 +--------+-------------------------+
 |  \*, / | multiplication, division|
 +--------+-------------------------+
 | +x,-x  | positive, negative      |
 +--------+-------------------------+
 |  \*\*  | exponentiation          |
 +--------+-------------------------+
 
Operator precedence increases from top to bottom and left to 
right (adapted from the `Python Reference Manual 
<https://docs.python.org/3/reference/expressions.html#operator-precedence>`_). 

The PySCeS MDL parser has been developed to parse and translate different
styles of infix into Python/Numpy-based expressions. The following
functions are supported in any mathematical expression:

  * ``log``, ``log10``, ``ln``, ``abs`` (note, *log* is defined as natural 
    logarithm, equivalent to *ln*)
  * ``pow``, ``exp``, ``root``, ``sqrt``
  * ``sin``, ``cos``, ``tan``, ``sinh``, ``cosh``, ``tanh``
  * ``arccos``, ``arccosh``, ``arcsin``, ``arcsinh``, ``arctan``, ``arctanh``
  * ``floor``, ``ceil``, ``ceiling``, ``piecewise``
  * ``notanumber``, ``pi``, ``infinity``, ``exponentiale``

Logical operators are supported in rules, events, etc., but *not*
in rate equation definitions. The PySCeS parser understands
Python infix as well as libSBML and NumPy prefix notation.  

  * ``and`` ``or`` ``xor`` ``not``
  * ``>`` ``gt(x,y)`` ``greater(x,y)``
  * ``<`` ``lt(x,y)`` ``less(x,y)``
  * ``>=`` ``ge(x,y)`` ``geq(x,y)`` ``greater_equal(x,y)``
  * ``<=`` ``le(x,y)`` ``leq(x,y)`` ``less_equal(x,y)``
  * ``==`` ``eq(x,y)`` ``equal(x,y)``
  * ``!=`` ``neq(x,y)`` ``not_equal(x,y)``

Note that currently the MathML *delay and factorial* functions 
are not supported. Delay is handled by simply removing it from 
any expression, e.g. *delay(f(x), delay)* would be parsed as 
*f(x)*. Support for *piecewise* has been recently added 
to PySCeS and will be discussed in the *advanced features* section (see 
:ref:`PySCeS-Inputfile-Advanced-Piecewise`). 

A reaction definition when no compartments are defined:

::

  R5: Fru + ATP = Hex_P + ADP
      Vmax5/(1 + Fru/Ki5_Fru)*(Fru/Km5_Fru)*(ATP/Km5_ATP) /
      (1 + Fru/Km5_Fru + ATP/Km5_ATP + Fru*ATP/(Km5_Fru*Km5_ATP) + ADP/Ki5_ADP)

and using the previously defined functions:  ::

  R6:
      A = B
      rmm_num(V2,A,B,Keq2)/rmm_den(A,B,K2A,K2B)

When compartments are defined, note how now the reaction is now 
given a location. This is because the ODEs formed from these 
reactions must be in changes in substance (amount) per time, thus the rate 
equation is multiplied by its compartment size. In this 
particular example the species symbols represent concentrations 
(*Species_In_Conc: True*):  :: 

  R1@Cell:
      s1 = s2
      Cell*(Vf1*(s1 - s2/Keq1)/(s1 + KS1*(1 + s2/KP1)))

If *Species_In_Conc: True* the location of the species is 
defined when it is initialised. 

The following example shows the species symbols 
explicitly defined as amounts (*Species_In_Conc: False*):: 

 R4@Memb: s3 = s4
     Memb*(Vf4*((s3/Memb) - (s4/Cell)/Keq4)/((s3/Memb)
     + KS4*(1 + (s4/Cell)/KP4)))

Please note that at this time we are not certain if this form 
of rate equation is translatable into valid SBML in a way that is 
interoperable with other software. 

.. _PySCeS-Inputfile-Detailed-Initialisation:

Species and parameter initialisation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The general form of any species (fixed, free) and parameter is 
simply: :: 

  property = value

Initialisations can be written in any order anywhere in the 
input file but for enhanced human readability these are usually 
placed after the reaction that uses them or grouped at the end 
of the input file. Both decimal and scientific notation are 
allowed with the following provisions that neither floating 
point (``1.``) nor scientific shorthand (``1.e-3``) syntax should 
be used, instead use the full form (``1.0e-3``, ``0.001`` or 
``1.0``). 

Variable or free species are initialised differently depending 
on whether compartments are present in the model or not. Although the
variable species concentrations are determined by 
the parameters of the system, their initial values are used in 
various places, calculating total moiety concentrations (if 
present), time simulation initial values (e.g. time=zero) and 
as initial guesses for the steady-state algorithms. If an empty 
initial species pool is required it is not recommended to 
initialise these values to zero (in order to prevent potential 
divide-by-zero errors) but rather to a small value (e.g. 
``1.0e-8``). 

For a model with no compartments these initial values are assumed 
to be concentrations:  :: 

  NADH = 0.001
  ATP  = 2.3e-3
  sucrose = 1
 
In a model with compartments it is expected that the species 
are located in a compartment (even if *Species_In_Conc: False*);
this is done using the *@* symbol:: 

 s1@Memb = 0.01
 s2@Cell = 2.0e-4

A word of warning, the user is responsible for making sure that 
the units of the initialised species match those of the model. 
Please keep in mind that **all** species (and anything that 
depends on them) are defined in terms of the *Species_In_Conc* 
keyword. For example, if the preceding initialisations were for 
*R1* (see Reaction section) then they would be concentrations 
(as *Species_In_Conc: True*). However, in the next example, we 
are initialising species for *R4* and they are therefore in 
amounts (*Species_In_Conc: False*).  :: 

  s3@Memb = 1.0
  s4@Cell = 2.0

Fixed species are defined in a similar way and although they
are technically parameters, they should be given a location in 
compartmental models:  :: 

 # InitExt
 X0 = 10.0
 X4@Cell = 1.0

However, fixed species are true parameters in the sense that 
their associated compartment size does not affect their value 
when it changes size. If compartment size-dependent behaviour 
is required, an assignment or rate rule should be considered. 

Finally, the parameters should be initialised. PySCeS checks if 
a parameter is defined that is not present in the rate 
equations and if such parameter initialisations are detected a 
harmless warning is generated. If, on the other hand, an 
uninitialised parameter is detected a warning is generated and 
a value of 1.0 assigned:  :: 

  # InitPar
  Vf2 = 10.0
  Ks4 = 1.0

.. _PySCeS-Inputfile-Advanced:

Advanced model construction
---------------------------

.. _PySCeS-Inputfile-Advanced-Assignment:

Assignment rules
~~~~~~~~~~~~~~~~

Assignment rules or forcing functions are used to set the value 
of a model attribute before the ODEs are evaluated. This model 
attribute can either be a parameter used in the rate equations 
(this is traditionally used to describe an equilibrium block), a 
compartment, or an arbitrary parameter (commonly used to define 
some sort of tracking function). Assignment rules can access 
other model attributes directly and have the generic form ``!F 
<par> = <formula>``, where *<par>* is the parameter assigned 
the result of *<formula>*. Assignment rules can be defined 
anywhere in the input file: :: 

  !F S_V_Ratio = Mem_Area/Vcyt
  !F sigma_test = sigma_P*Pmem + sigma_L*Lmem
 
These rules would set the value of *<par>*, whose value 
can be followed using the simulation and steady state 
*extra_output* functionality (see :ref:`Simulation_Results` and 
:ref:`Steady_state_data_object`). 

.. _PySCeS-Inputfile-Advanced-Raterule:

Rate rules
~~~~~~~~~~

PySCeS includes support for rate rules, which are 
essentially directly encoded ODEs that are evaluated after 
the ODEs defined by the model stoichiometry and rate 
equations. Unlike the SBML rate rule, PySCeS allows one to 
directly access a reaction symbol in the rate rules (this is 
automatically expanded when the model is exported to SBML). The 
general form of a rate rule is ``RateRule: <name> 
{<formula>}``, where *<name>* is the model attribute (e.g. 
compartment or parameter) whose rate of change is described by 
the *<formula>*. It may also be defined anywhere in the input 
file:  :: 

  RateRule: Mem_Area {
  (sigma_P)*(Mem_Area*k4*(P)) + (sigma_L)*(Mem_Area*k5*(L))
  }

  RateRule: Vcyt {(1.0/Co)*(R1()+(1-m1)*R2()+(1-m2)*R3()-R4()-R5())}

Remember to initialise any new parameters defined in the rate rules.
 
.. _PySCeS-Inputfile-Advanced-Events:

Events
~~~~~~

Time-dependent events may be defined. Since PySCeS 1.1.0 their definition 
follows the event framework described in the SBML L3V2 
specification. The general form of an event is:

``Event: <name>, <trigger>, <optional_kwargs> { <assignments> }``

As can be seen, an event consists of essentially two parts, a conditional 
*<trigger>*, and a set of one or more *<assignments>*. Assignments have the 
general form ``<par> = <formula>``. Events have access to the "current" 
simulation time using the ``_TIME_`` symbol:: 

  Event: event1, _TIME_ > 10 and A > 150.0 {
  V1 = V1*vfact
  V2 = V2*vfact
  }

.. note::

  In order for PySCeS to handle events it is necessary to
  have Assimulo installed (refer to :ref:`General_Requirements`).

In line with the SBML L3V2 specification, three **optional keyword arguments** 
can be defined as a comma-separated list following the trigger definition. The 
general syntax is ``<attribute>=<value>``. The keywords are:

* *delay* (float): specifies a delay between 
  when the trigger is fired (and the assignments are evaluated) 
  and the eventual assignment to the model. If this keyword is not specified, a 
  value of ``0.0`` is assumed.
* *priority* (integer or None): specifies a priority for events that trigger at 
  the same simulation time. Events with a higher priority are executed before 
  those with a lower priority, while events without a priority (``None``) are 
  executed in random positions in the sequence. If this keyword is not 
  specified, a value of ``None`` is assumed.
* *persistent* (boolean): is only relevant to events with a delay, where the 
  situation may occur that the trigger condition no 
  longer holds by the time the delay in the simulation has passed. The 
  *persistent* attribute specifies how to deal with this situation: if 
  ``True``, the event executes nevertheless; if ``False``, the event does not 
  execute if the trigger condition is no longer valid. If the keyword is not 
  specified, a default of ``True`` is assumed.

The reader is referred to the `SBML Specification 
<https://sbml.org/documents/specifications>`_ for further details.

The following event illustrates the use of a delay of ten time units with a 
non-persistent trigger and a priority of 3. In addition, the prefix notation 
(used by libSBML) for the trigger is illustrated (PySCeS understands both 
notations):  :: 

  Event: event2, geq(_TIME_, 15.0), delay=10.0, persistent=False, priority=3 {
  V3 = V3*vfact2
  } 
  
.. note::

  #. The **legacy event specification** (PySCeS versions <1.1), which did not 
     include keywords for `priority` and `persistent`, and in which the 
     delay was specified as a third positional argument (without 
     keyword), is still supported but deprecated:

     ``Event: <name>, <trigger>, <delay> { <assignments> }``
     
  #. The following SBML event attributes are **not implemented**:
  
     * ``event.use_values_from_trigger_time=False`` For an event with a 
       delay, PySCeS always uses the assignment values from the time 
       when the event is triggered. When loading an SBML model 
       (see :ref:`PySCeS-Inputfile-Advanced-SBML`) that uses the 
       assignment values from the time of event firing and thus has this 
       event attribute set, a `NotImplementedError` is raised.
       
     * ``trigger.initial_value=False`` PySCeS always assumes that the initial 
       value of a trigger is `True`, i.e. the event cannot fire at time zero, 
       but that the simulation has to run for at least one iteration before the 
       trigger can be fired. When loading and SBML model that has the initial 
       value set to `False`, a `NotImplementedError` is raised. 

.. _PySCeS-Inputfile-Advanced-Piecewise:

Piecewise
~~~~~~~~~

Although technically an operator, piecewise functions are 
sufficiently complicated to warrant their own section. A 
piecewise operator is essentially an *if, elif, ..., else* 
logical operator that can be used to conditionally "set" the 
value of some model attribute. Currently piecewise is supported 
in rule constructs and has not been tested directly in rate 
equation definitions. The piecewise function's most basic 
incarnation is ``piecewise(<val1>, <cond>, <val2>)``, which is 
evaluated as:  

.. code-block:: python

  if <cond>:
      return <val1>
  else:
      return <val2>

Alternatively, 
``piecewise(<val1>, <cond1>, <val2>, <cond2>, <val3>, <cond3>)``

.. code-block:: python

  if <cond1>:
      return <val1>
  elif <cond2>:
      return <val2>
  elif <cond3>:
      return <val3>

Or ``piecewise(<val1>, <cond1>, <val2>, <cond2>, <val3>, <cond3>, <val4>)``

.. code-block:: python

  if <cond1>:
      return <val1>
  elif <cond2>:
      return <val2>
  elif <cond3>:
      return <val3>
  else:
      return <val4>

can also be used. A "real-life" example of an assignment rule 
with a piecewise function:  :: 

  !F Ca2plus=piecewise(0.1, lt(_TIME_,60), 0.1, gt(_TIME_,66.0115), 1)  

In principle there is no limit on the number of conditional 
statements present in a piecewise function; the condition can 
be a compound statement (``a or b and c``) and may include the 
``_TIME_`` symbol. 

Reagent placeholder
~~~~~~~~~~~~~~~~~~~

Some models contain reactions that are defined as only having substrates or
products, with the fixed (external) species not specified:  ::

  R1: A + B >

  R2: > C + D
 
The implication is that the relevant reagents appear from or disappear into
a constant pool. Unfortunately the `PySCeS` parser does not accept
such an unbalanced reaction definition and requires these pools to be
represented with a ``$pool`` token::

 R1: A + B > $pool
 
 R2: $pool > C + D

``$pool`` is neither counted as a reagent nor does it ever appear in the
stoichiometry (think of it as *dev/null*) and no other ``$<str>`` tokens are 
allowed.

.. _PySCeS-Inputfile-Advanced-SBML:

SBML import and export
~~~~~~~~~~~~~~~~~~~~~~

SBML models can be imported into PySCeS by first converting them to the input file
(``*.psc``) format and then loading the input file into PySCeS as usual. 
The conversion is done with the ``pysces.interface.convertSBML2PSC()`` method: ::

  >>> pysces.interface.convertSBML2PSC(sbmlfile, sbmldir=None, pscfile=None, pscdir=None)

where:

- *sbmlfile*: the SBML filename
- *sbmldir*: the directory of SBML files (if `None`, the current working directory is 
  assumed)
- *pscfile*: the output PSC file name (if `None`, ``<sbmlfile>.psc`` is used)
- *pscdir*: the PSC output directory (if `None`, the ``pysces.model_dir`` is used)

An instantiated PySCeS model can be exported to SBML using the 
``pysces.interface.writeMod2SBML()`` method: ::

  >>> pysces.interface.writeMod2SBML(mod, filename=None, directory=None, iValues=True, getdocument=False, getstrbuf=False)

where:

- *mod*: is the PySCeS model object
- *filename*: writes ``<filename>.xml`` or ``<model_name>.xml`` if `None`
- *directory*: (optional) an output directory
- *iValues*: if `True`, the model initial values are used 
  (or the current values if `False`)
- *getdocument*: if `True` an SBML document object is returned instead 
  of writing to disk
- *getstrbuf*: if `True` a ``StringIO`` buffer is returned instead of writing to disk


.. _PySCeS-Inputfile-Examples:

Example PySCeS input files
--------------------------

.. _PySCeS-Inputfile-Examples-Basic:

Basic model definition
~~~~~~~~~~~~~~~~~~~~~~

PySCeS test model: *pysces_test_linear1.psc* - this file is distributed with 
PySCeS and copied to your model directory (typically *$HOME/Pysces/psc*) after 
installation, when running ``pysces.test()`` for the first time.

.. code-block:: python

  FIX: x0 x3

  R1: x0 = s0
      k1*x0 - k2*s0

  R2: s0 = s1
      k3*s0 - k4*s1

  R3: s1 = s2
      k5*s1 - k6*s2

  R4: s2 = x3
      k7*s2 - k8*x3

  # InitExt
  x0 = 10.0
  x3 = 1.0
  # InitPar
  k1 = 10.0
  k2 = 1.0
  k3 = 5.0
  k4 = 1.0
  k5 = 3.0
  k6 = 1.0
  k7 = 2.0
  k8 = 1.0
  # InitVar
  s0 = 1.0
  s1 = 1.0
  s2 = 1.0

.. _PySCeS-Inputfile-Examples-Advanced:

Advanced example
~~~~~~~~~~~~~~~~

This model includes the use of *Compartments*, *KeyWords*, 
*Units* and *Rules*:

.. code-block::

  Modelname: MWC_wholecell2c
  Description: Surovtsev whole cell model using J-HS Hofmeyr's framework

  Species_In_Conc: True
  Output_In_Conc: True

  # Global unit definition
  UnitVolume: litre, 1.0, -3, 1
  UnitSubstance: mole, 1.0, -6, 1
  UnitTime: second, 60, 0, 1

  # Compartment definition
  Compartment: Vcyt, 1.0, 3
  Compartment: Vout, 1.0, 3
  Compartment: Mem_Area, 5.15898, 2

  FIX: N 

  R1@Mem_Area: N = M
    Mem_Area*k1*(Pmem)*(N/Vout)

  R2@Vcyt: {244}M = P # m1
    Vcyt*k2*(M)

  R3@Vcyt: {42}M = L # m2
    Vcyt*k3*(M)*(P)**2

  R4@Mem_Area: P = Pmem
    Mem_Area*k4*(P)

  R5@Mem_Area: L = Lmem
    Mem_Area*k5*(L)

  # Rate rule definition
  RateRule: Vcyt {(1.0/Co)*(R1()+(1-m1)*R2()+(1-m2)*R3()-R4()-R5())}
  RateRule: Mem_Area {(sigma_P)*R4() + (sigma_L)*R5()}

  # Rate rule initialisation
  Co = 3.07e5 # uM p_env/(R*T)
  m1 = 244
  m2 = 42 
  sigma_P = 0.00069714285714285711
  sigma_L = 0.00012

  # Assignment rule definition
  !F S_V_Ratio = Mem_Area/Vcyt
  !F Mconc = (M)/M_init
  !F Lconc = (L)/L_init
  !F Pconc = (P)/P_init

  # Assignment rule initialisations
  M_init = 199693.0
  L_init = 102004
  P_init = 5303
  Mconc = 1.0
  Lconc = 1.0
  Pconc = 1.0

  # Species initialisations
  N@Vout = 3.07e5
  Pmem@Mem_Area = 37.38415
  Lmem@Mem_Area = 8291.2350678770199
  M@Vcyt = 199693.0
  L@Vcyt = 102004
  P@Vcyt = 5303

  # Parameter initialisations
  k1 = 0.00089709
  k2 = 0.000182027
  k3 = 1.7539e-010
  k4 = 5.0072346e-005
  k5 = 0.000574507164

  """
  Simulate this model to 200 for maximum happiness and
  watch the surface to volume ratio and scaled concentrations.
  """
 
This example illustrates almost all of the features included 
in the PySCeS MDL. Although it may be slightly more complicated 
than the basic model described above it is still, by our 
definition, human readable. 



.. _SBML:       http://sbml.org
