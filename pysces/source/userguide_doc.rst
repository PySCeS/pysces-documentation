.. _Introducing-PySCeS:

Introduction
============

 PySCeS: the Python Simulator for Cellular Systems is an extendable toolkit for
 the analysis and investigation of cellular systems. PySCeS is available for
 download at http://pysces.sourceforge.net and on GitHub where the source
 code is maintained: https://github.com/pysces

Welcome! This users guide will get you started with the basics of modelling
cellular systems with PySCeS. It is meant to be used together with the *input
file guide*. If you already have PySCeS installed continue straight on
if not, :ref:`Installing-PySCeS` contains instructions on building and
installing PySCeS.

 PySCeS is distributed under the PySCeS (BSD style) license and is made
 freely available as Open Source software. See LICENCE.txt for details.

PySCeS continued development depends, to a large degree on 
support and feedback from Systems Biology community, 
if you use PySCeS in your work please cite it using the 
following reference: 

 Brett G. Olivier, Johann M. Rohwer and Jan-Hendrik S. Hofmeyr
 *Modelling cellular systems with PySCeS*, Bioinformatics, 21, 560-561,
 DOI 10.1093/bioinformatics/bti046.
     
We hope that you will enjoy using our software. If, however, you find any
unexpected features (i.e. bugs) or have any suggestions on how we can improve
PySCeS please let us know.


**The PySCeS development team.**


.. _Running-PySCeS:

Getting started
===============

Loading PySCeS
--------------

In this section we assume you have PySCeS installed and 
configured (see `Installing and configuring`_ for details) and a 
correctly formatted PySCeS input file that describes a cellular 
system in terms of its reactions, species and parameters. For a 
detailed description of the PySCeS Model Description Language 
see the *PySCeS Input File Guide*. Note that on all platforms 
PySCeS model files have the extension ``.psc``. 

To begin modelling we need to start up an interactive Python shell
(we suggest iPython_) and load PySCeS with ``import pysces``::

 Python 2.7.11 |Anaconda 2.4.1 (64-bit)| (default, Dec  7 2015, 14:10:42) [MSC v.1500 64 bit (AMD64)]
 Type "copyright", "credits" or "license" for more information.
 
 In [1]: import pysces
 Matplotlib backend set to: "TkAgg"
 Matplotlib interface loaded (pysces.plt.m)
 Continuation routines available
 NLEQ2 routines available
 You are using NumPy (1.10.1) with SciPy (0.16.0)
 RateChar is available
 Parallel scanner is available
 
 PySCeS environment
 ******************
 pysces.model_dir = d:\\mypysces\\pscmodels
 pysces.output_dir = c:\\Pysces


 ***********************************************************************
 * Welcome to PySCeS (0.9.7) - Python Simulator for Cellular Systems   *
 *                http://pysces.sourceforge.net                        *
 * Copyright(C) B.G. Olivier, J.M. Rohwer, J.-H.S. Hofmeyr, 2004-2019  *
 * Triple-J Group for Molecular Cell Physiology                        *
 * Stellenbosch University, ZA and VU University Amsterdam, NL         *
 * PySCeS is distributed under the PySCeS (BSD style) licence, see     *
 * LICENCE.txt (supplied with this release) for details                *
 * Please cite PySCeS with: doi:10.1093/bioinformatics/bti046          *
 ***********************************************************************
 
PySCeS is now ready to use. If you would like to test your 
installation try running the test suite::

 pysces.test()
 
this also copies the test models supplied with PySCeS into your 
model directory. 


Creating a PySCeS model object
------------------------------

 This guide uses the test models supplied with PySCeS as 
 examples, if you would like to use them and have not already 
 done so run the PySCeS tests (described in the previous 
 section). 

Before modelling, a PySCeS model object needs to be instantiated.
As a convention we use ``mod`` as the instantiated model
instance. The following code creates such an instance using the
test input file, ``pysces_test_linear1.psc``::

 >>> mod = pysces.model('pysces_test_linear1')
 
 Assuming extension is .psc
 Using model directory: C:\mypysces\pscmodels
 C:\mypysces\pscmodels\pysces_test_linear1.psc loading .....

 Parsing file: C:\mypysces\pscmodels\pysces_test_linear1.psc
 Calculating L matrix . . . . . . .  done.
 Calculating K matrix . . . . . . .  done.

When instantiating a new model object, PySCeS input files are 
assumed to have a ``.psc`` extension. If the specified input 
file does not exist in the input file directory (e.g. 
misspelled filename) a list of existing input files is shown 
and the user is given an opportunity to enter the correct 
filename. 

Advanced 
~~~~~~~~ 

The model constructor can also be used to specify a model 
directory other than the default model path:: 

 mod = pysces.model('pysces_test_linear1', dir='c:\\Pysces\\psc')

alternatively input files can also be loaded from a string::

 >>> F = file('c:\\Pysces\\psc\\pysces_test_linear1.psc', 'r')
 >>> pscS = F.read()
 >>> F.close()
 >>> mod = pysces.model('test_lin1s', loader='string', fString=pscS)
 
 Assuming extension is .psc
 Using model directory: C:\mypysces\pscmodels
 Using file: test_lin1s.psc
 C:\mypysces\pscmodels\orca\test_lin1s.psc loading .....

note that now the input file is saved and loaded as 
``model_dir\orca\test_lin1s.psc``. 

Creating a PySCeS model object
------------------------------

Once a new model object has been created it needs to be loaded. 
During the load process the input file is parsed, the model 
description is translated into Python data structures and a 
stoichiometric structural analysis is performed::
 
 **New** in PySCeS 0.7.1+ model loading is now automatically performed
 when the model object is instantiated. This behaviour is controlled
 by the *autoload* argument (default = True). To keep backwards compatibility
 with older modelling scripts the first time ``doLoad()`` is called a warning
 is generated. Subsequent calls to ``doLoad()`` will reload the model as required.
 
Once loaded, all the model elements contained in the input file 
are made available as model (``mod``) attributes so that in the 
input file where you might find initialisations such as ``s1 = 
1.0`` and ``k1 = 10.0`` these are now available as ``mod.s1`` 
and ``mod.k1``. For variable species and compartments an 
additional attribute is created which contains the elements 
*initial* (as opposed to current) value. These are constructed as
<name>*_init*:: 

 >>> mod.s1
 1.0
 >>> mod.s1_init
 1.0
 >>> mod.k1
 10.0

Any errors generated during the loading process (almost always) 
occur as a result of syntax errors in the input file. These 
error messages may not be intuitive for example, \'list out of 
range\' exception usually indicates a missing multiplication 
operator \"3(\" instead of \"3*(\" or unbalanced parentheses. 

Basic model attributes
----------------------

Some basic model properties are accessible once the model is
loaded:

* ``mod.ModelFile``, the name of the model file that was used.

* ``mod.ModelDir``, the input file directory.

* ``mod.ModelOutput``, the PySCeS work/output directory.

* Parameters are available as attributes directly as specified 
  in the input file e.g. ``k1`` is ``mod.k1``.

* External (fixed) species are made available in the same way.

* Internal (variable) species are treated in a similar way except that an
  additional attribute (parameter) is created to hold the species initial value
  (as specified in the input file), e.g., from ``s1``, ``mod.s1`` and
  ``mod.s1_init`` are instantiated as model object attributes.

* Compartments are also are assigned an initial value.

* Rate equations are translated into objects that return their current value
  when called ``mod.R1()``.

All basic model attributes that are described here can be 
changed interactively. However, if the model rate equations need 
to be changed, this should be done in the input file after 
which the model should be re-instantiated and reloaded. 

Groups of model properties (either tuples, lists or dictionaries)

* ``mod.species`` the model's variable species names (ordered 
  relative to the stoichiometric matrix rows). 

* ``mod.reactions`` reaction names and ordered to the stoichiometric matrices columns. 

* ``mod.parameters`` all parameters (including fixed species)

* ``mod.fixed_species`` only the fixed species names

* ``mod.__rate_rules__`` a list of rate rules defined in the model  

Advanced
~~~~~~~~

These attributes are used by PySCeS to store additional 
information about the basic model components, generally they 
are supplied by the parser and should almost never be changed 
directly. 

* ``mod.__events__`` a list of event object references 
  which can be interrogated for event information. For example if you 
  want a list of event names try ``[ev.name for ev in mod.__events__]``

* ``mod.__rules__`` a dictionary containing information about all rules defined for this model
   
* ``mod.__sDict__`` a dictionary of species information

* ``mod.__compartments__`` a dictionary containing compartment information   

.. _Core_Analysis:

Modelling
=========

Structural Analysis
-------------------

As part of the model loading procedure, ``doLoad()`` automatically performs
a stoichiometric (structural) analysis of the model. The structural
properties of the model are captured in stoichiometric matrix (**N**),
kernel matrix (**K**) and link matrix (**L**). These matrices can
either be displayed with a ``mod.showX()`` method or used in further
calculations as numeric arrays. The formal definition of these matrices,
as they are used in PySCeS, is described in [#]_.

The structural properties of a model are available in two 
forms, as new-style objects which have all the array properties 
neatly encapsulated or as legacy attributes. Although both 
exist it is highly recommended to use the new objects. 

Structural Analysis - new objects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For alternate descriptions of these model properties see the 
next (legacy) section. 

* ``mod.Nmatrix`` view with ``mod.showN()`` 

* ``mod.Nrmatrix`` view with ``mod.showNr()``

* ``mod.Lmatrix`` view with ``mod.showK()``

* ``mod.L0matrix``

* ``mod.Kmatrix`` view with ``mod.showL()``

* ``mod.K0matrix``

* ``mod.showConserved()`` displays any moiety conserved relationships (if present).

* ``mod.showFluxRelationships()`` shows the relationships 
  between dependent and independent fluxes at steady state. 

All new structural objects have an *array* attribute which 
holds the actual NumPy array data as well as *ridx* and *cidx* 
which hold the row and column indexes (relative to the 
stoichiometric matrix) as well as the following methods: 

* ``.getLabels()`` return the matrix labels as tuple([rows], [columns])

* ``.getColsByName()`` extract column(s) with label

* ``.getRowsByName()`` extract row(s) with label

* ``.getIndexes()`` return the matrix indices (relative to the
  Stoichiometric matrix) as tuple((rows), (columns))

* ``.getColsByIdx()`` extract column(s) referenced by index

* ``.getRowsByIdx()`` extract row(s) referenced by index


Structural Analysis - legacy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``mod.nmatrix``, **N**: displayed with ``mod.showN()``

* ``mod.kmatrix``, **K**: displayed with ``mod.showK()``

* ``mod.lmatrix``, **L**: displayed with ``mod.showL()`` (an identity
  matrix means **L** does not exist i.e. no linear dependence).

* If there are linear dependencies in the differential equations then the
  reduced stoichiometric matrix of linearly independent, differential
  equations **Nr** is available as ``mod.nrmatrix`` and is displayed with
  ``mod.showNr()``. If there is no dependence **Nr** = **N**.

* In the case where there is linear dependence the moiety conservation sums
  can be displayed by using ``mod.showConserved()``. The conservation totals
  are calculated from the initial values of the variable species as defined
  in the model file.

* When the **K** and **L** matrices exist, their dependent parts
  (**K0**, **L0**) are available as ``mod.kzeromatrix`` and
  ``mod.lzeromatrix``.

* ``mod.showConserved()`` prints any moiety conserved relationships (if
  present).

* ``mod.showFluxRelationships()`` shows the relationships between dependent
  and independent fluxes at steady state.

If the ``mod.showX()`` methods are used the row and column titles of the
various matrices are displayed with the matrix. Additionally, all of the
``mod.showX()`` methods accept an open file object as an argument. If this
file argument is present the method's results are output to a file and not
printed to the screen. Alternatively, the order of each matrix dimension,
relative to the stoichiometric matrix, is available as either a row or
column array (e.g., ``mod.krow``, mod.lrow``, ``mod.kzerocol``).

Time simulation
---------------

PySCeS has interfaces to two ODE solvers either LSODA from 
ODEPACK (part of SciPy) or SUNDIALS CVODE (using PySundials). 
If PySundials is installed it will automatically select CVODE 
if compartments, events or rate rules are detected during model 
load as LSODA is not able capable of event handling or changing 
compartment sizes. If, however, you would like to select the 
solver manually this is also possible:: 

 mod.mode_integrator = 'LSODA'
 mod.mode_integrator = 'CVODE'

There are three ways of running a simulation:

1. Defining the *start*, *end* time and number of *points* and using the
   ``mod.Simulate()`` method directly::
     
     mod.sim_start = 0.0
     mod.sim_end = 20
     mod.sim_points = 50
     mod.Simulate()

2. Using the ``mod.doSim()`` method where only the *end* time and *points*
   need to be specified. For example running a 20 point simulation from time
   0 to 10::

     >>> mod.doSim(end=10.0, points=20.0)

3. Or using ``mod.doSimPlot()`` which runs the simulation and displays the
   results. In addition to doSim's arguments the following arguments may
   be used::
   
    mod.doSimPlot(end=10.0, points=21, plot='species', fmt='lines', filename=None)

   where: 
   
   - *plot* can be one of species, rates or all.
   - *fmt* plot format, UPI backend dependent (default='') or the *CommonStyle* 'lines' or 'points'.
   - *filename* if not None (default) then the plot is exported as *filename*.png

Another way of quickly visualising the results of a simulation 
is to use the ``mod.SimPlot`` method::

 mod.SimPlot(plot='species', filename=None, title=None, log=None, format='lines')

where:

- *plot*: output to plot (default='species')

 - 'all' rates and species
 - 'species' species
 - 'rates' reaction rates
 - `['S1', 'R1', ]` a list of model attributes (species, rates)

- *filename* (optional) if not None file is exported to filename (default=None)
- *title* the plot title (default=None)
- *log* use log axis for 'x', 'y', 'xy' (default=None)
- *fmt* plot format, UPI backend dependent (default='') or the *CommonStyle* 'lines' or 'points'.

Called without arguments, ``mod.SimPlot()`` plots all the species
concentrations against time. 

Simulation results
~~~~~~~~~~~~~~~~~~

In PySCeS 0.7.x the simulation results have been consolidated 
into a new ``mod.data_sim`` object. By default species 
concentrations/amounts, reaction rates and rate rules are 
automatically added to the *data_sim* object. If extra 
information (parameters, compartments, assignment rules) is 
required this can easily be added using ``mod.CVODE_extra_output``, a
list containing any model attribute which is not added by default.

The ``mod.data_sim`` object which has many methods for extracting simulation
data including:

* ``data_sim.getTime()`` return a vector of time points

* ``data_sim.getSpecies()`` returns array([[time], [species]])

* ``data_sim.getRates()`` returns array([[time], [rates]])

* ``data_sim.getRules()`` returns array([[time], [rate rules]])

* ``data_sim.getXData`` returns array([[time], [CVODE_extra_output]])

* ``data_sim.getSimData(*args)`` return an array consisting of *time* plus any
  available data series:: 
  
   mod.data_sim.getSimdata('s1', 'R1', 'Rule1', 'xData2')

* ``data_sim.getAllSimData(*args)`` return an array of all simulation data

* ``data_sim.getDataAtTime(time)`` return the results of the simulation at
  *time*.

* ``data_sim.getDataInTimeInterval(time, bound)`` return the simulation
  data in the interval [time-bound, time+bound], if *bound* is not
  specified it is assumed to be the step size.

All the *data_sim.get\** methods by default only return a NumPy array containing
the requested data, however if the argument *lbls* is set to True then both
the array as well as a list of column labels is returned::

 Sdata, Slabels = mod.data_sim.getSpecies(lbls=True)

This is very useful when using the PySCeS plotting interface 
(described later in this guide) to plot simulation results. 

Advanced
~~~~~~~~

PySCeS sets integrator options that attempt to configure the integration
algorithms to suit a particular model. However, almost every integrator
option can be overridden by the user. 
Simulator settings are stored in PySCeS ``mod.__settings__`` 
dictionary. For LSODA some useful keys are
(mod.__settings__[*key*]):: 

 'lsoda_atol': 1.0e-012
 'lsoda_rtol': 1.0e-007
 'lsoda_mxordn': 12
 'lsoda_mxords': 5
 'lsoda_mxstep': 0

atol and rtol are the absolute and relative tolerances, while mxstep=0
means that LSODA chooses the number of steps (up to 500). If this is
still not enough, PySCeS automatically increases the number of steps
necessary to find a solution.   

Additionally, CVODE allows per step step-size optimisation and automatic 
tolerance scaling:: 

 'cvode_abstol': 1.0e-15
 'cvode_abstol_factor': 1.0e-8
 'cvode_auto_tol_adjust': True
 'cvode_mxstep': 1000
 'cvode_reltol': 1.0e-9
 'cvode_stats': False

*cvode_abstol* is considered to be the minimum absolute 
tolerance, PySCeS first uses the initial species values 
multiplied by *cvode_abstol_factor* (so that [s]*[factor] >= [abstol]) 
to calculate its absolute tolerance. Once the simulation is 
underway PySCeS periodically readjusts the absolute tolerance 
on a per species basis based on the current species value.

If CVODE cannot find a solution in the given number of steps it 
automatically increases *cvode_mxstep* and tries again, 
however, it also keeps track of the number of times that this 
adjustment is required and if a specific threshold is passed it 
will begin to increase *cvode_reltol* by 1.0e3 (to a maximal 
value of 1.0e-3). Finally, if *cvode_stats* is enabled CVODE will 
display a report of its internal parameters after the 
simulation is complete. 


Steady-state analysis
---------------------

PySCeS solves for a steady state using either the non-linear solvers
HYBRD_,  NLEQ2_ or forward integration. By default PySCeS has *solver
fallback* enabled which means that if a solver fails or returns an invalid
result (i.e., contains negative concentrations) it switches to the next
available solver. The solver chain is as follows: 

1. HYBRD (can handle 'rough' initial conditions, converges quickly).

2. NLEQ2 (highly optimised for extremely non-linear systems, 
   more sensitive to bad conditioning and slightly slower convergence).

3. FINTSLV (finds a result when the change in max([species]) is less than 0.1%;
   slow convergence).

Solver fallback can be disabled by setting ``mod.mode_solver_fallback =
0``. Each of the three solvers is highly configurable and although the
default settings should work for most models configurable options
can be set in by way of the *mod.__settings__* dictionary.

To calculate a steady state use the ``mod.doState()`` method::

  >>> mod.doState() 
  (hybrd) The solution converged.

The results of a steady-state evaluation are stored as arrays as well as
individual attributes and can be easily displayed using the
``mod.showState()`` method:

* ``mod.showState()`` displays the current steady-state values of both the
  species and fluxes.

* For each reaction (e.g. ``R2``) a new attribute ``mod.J_R2``, which
  represents its steady-state value, is created.

* Similarly, each species (e.g. ``mod.s2``) has a steady-state attribute
  ``mod.s2_ss``

* ``mod.state_species`` in ``mod.species`` order.

* ``mod.state_flux`` in ``mod.reactions`` order.

There are various ways of initialising the steady-state solvers although,
in general, the default values can be used.

* ``mod.mode_state_init`` initialises the solver using either the initial
  values (0), a value close to zero (1). The default behaviour 
  is to use the initial values. 

New: mod.data_sstate
~~~~~~~~~~~~~~~~~~~~

New to PySCeS 0.7 is the ``mod.data_sstate`` object that by 
default stores steady-state data (species, fluxes, rate rules) 
in a manner similar to mod.data_sim. One notable exception is 
that the current steady-state values are also made available as 
attributes to this object (e.g. species S1's steady-state value 
is stored as ``mod.data_sstate.S1``). Using the 
``mod.STATE_extra_output`` list it is possible to store user 
defined data in the *data_sstate* object. Steady-state data can be
easily retrieved using the by now familiar *.get\** methods. 

- ``data_sstate.getSpecies()`` returns a species array
- ``data_sstate.getFluxes()`` returns a flux array       
- ``data_sstate.getRules()`` returns a rate rule array
- ``data_sstate.getXData()`` returns an array defined in *STATE_extra_output*       
- ``data_sstate.getStateData(*args)`` return user defined array of data ('S1','R2')
- ``data_sstate.getAllStateData()`` return all state data as an array 

All these methods also accept the lbls=True argument in which case they return both
array data and a label list::

 ssdat, sslbl = mod.data_sstate.getSpecies(lbls=True)

Stability
~~~~~~~~~

PySCeS can analyse the stability of systems that can attain a steady state.
It does this by calculating the Eigen values of the Jacobian matrix for the 
reduced system of independent ODE's::

- ``mod.doEigen()`` calculates a steady-state and performs the stability analysis
- ``mod.showEigen`` prints out a stability report
- ``mod.doEigenShow()`` combines both of the above

The Eigen values are also available as attributes 
``mod.lambda1`` etc. By default the Eigen values are stored as 
``mod.eigen_values`` but if 
``mod.__settings__['mode_eigen_output'] = 1`` is set both the 
Eigen values as well as the left and right Eigen vectors are 
stored as ``mod.eigen_vecleft`` and ``mod.eigen_vecright`` 
respectively. Please note that there is currently no guarantee 
that the order of the Eigen value array corresponds to the 
species order. 


Metabolic Control Analysis
--------------------------

For practical purposes the following methods are collected into a set of
meta-routines that all first solve for a steady state and then the required
Metabolic Control Analysis (MCA) [#]_, [#]_ evaluation methods.


Elasticities
~~~~~~~~~~~~

The elasticities towards both the variable species and parameters can be
calculated using ``mod.doElas()`` which generates as output:

* Scaled elasticities generated as ``mod.ecRate_Species``, e.g.
  ``mod.ecR4_s2``

* ``mod.showEvar()`` displays the non-zero elasticities calculated with
  respect to the variable species.

* ``mod.showEpar()`` displays the non-zero parameter elasticities.

As a prototype we also store the elasticities in an object, 
``mod.ec.*`` this may become the default way of accessing 
elasticity data in future releases but has not been stabilised 
yet. 

Control coefficients
~~~~~~~~~~~~~~~~~~~~

Both control coefficients and elasticities can be calculated using a single
method, ``mod.doMca()``.

* ``mod.showCC()`` displays the complete set of flux and concentration
  control coefficients.

* Individual control coefficients are generated as either
  ``mod.ccSpecies_Rate`` for a concentration control coefficient, e.g.
  ``mod.ccs1_R4``.

* Similarly, ``mod.ccJFlux_Rate`` is a flux control coefficient e.g.
  ``mod.ccJR1_R4``.

As it is generally common practice to use scaled elasticities 
and control coefficients PySCeS calculated these by default. 
However, it is possible to generate unscaled elasticities and 
control coefficients by setting the attribute 
``mod.__settings__['mode_mca_scaled'] = 0`` in which case the 
model attributes are attached as ``mod.uec`` and ``mod.ucc`` 
respectively. 

As a prototype we also store the elasticities in an object, 
``mod.cc.*`` this may become the default way of accessing 
control coefficient data in future releases but has not been 
stabilised yet. 

Response coefficients
~~~~~~~~~~~~~~~~~~~~~

A new PySCeS feature is the ability to calculate the parameter response
coefficients for a model with the ``mod.doMcaRC()`` method. Unlike the
elasticities and control coefficients the response coefficients are made
available as a single attribute ``mod.rc``. This attribute is a data
object, containing the response coefficients as attributes and has the
following methods:

* ``rc.var_par`` individual response coefficients can be accessed as
  attributes made up of ``variable_parameter`` e.g. ``mod.rc.R1_k1``

* ``rc.get('var', 'par')`` return a response coefficient

* ``rc.list()`` returns all response coefficients as a dictionary of
  {key:value} pairs

* ``rc.select('attr', search='a')`` select all response coefficients that
  refer to ``'attr'`` e.g. ``select('R1')`` or ``select('k2')``

* ``rc.matrix``: the matrix of response coefficients

* ``rc.row``: row labels

* ``rc.col``: column labels

Responce coefficients with respect to moiety-conserved sums
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``mod.doMcaRC()`` method only calculates response coefficients with respect to explicit model parameters. However, in models with moiety-conservation the total concentration of all the species that form part of a particular moiety-conserved cycle is also a parameter of the model. PySCeS infers such moiety-conserved sums from the initial species concentrations specified by the user. In some cases it might be interesting to consider the effects that a change in the total concentration of a moiety will have on the steady-state. This analysis may be done with the method ``mod.doMcaRCT()``.

Since moiety-conserved sums are not explicitly named in PySCeS model files, ``'T_'`` is prepended to all the species names listed in ``mod.Consmatrix.row``. For instance, if the dependent species in a moiety-conserved cycle is ``'A'``, then ``'T_A'`` designates the moiety-conserved sum.

The object ``mod.rc`` is augmented with the results of ``mod.doMcaRCT()``. Response coefficients may thus be accessed with ``mod.rc.get('var', 'T_par')``.


.. _Analysis:


Parameter scanning
==================

Single dimension parameter scans
--------------------------------

PySCeS has the ability to quickly generate and plot single dimension
parameter scans. Scanning a parameter typically involves changing a
parameter through a range of values and recalculating the steady state at
each step. Two methods are provided which simplify this task,
``mod.Scan1()`` is provided to generate the scan data while
``mod.Scan1Plot()`` is used to visualise the results. The first step is to
define the scan parameters:

* ``mod.scan_in`` is a string defining the parameter to be scanned e.g.
  ``'x0'``

* ``mod.scan_out`` is a list of strings representing the attribute names
  you would like to track in the output eg.
  ``['J_R1','J_R2','s1_ss','s2_ss']``

* You also need to define the range of points that you would like to scan
  over. For a linear range SciPy has a useful function
  ``scipy.linspace(start, end, points)`` (SciPy can be accessed by typing
  ``import scipy`` in your Python shell). If you need to generate a log range
  use ``scipy.logspace(start, end, points)``.

  Both ``scipy.linspace`` and ``scipy.logspace`` use the number of points
  (including the start and end points) in the interval as an input.
  Additionally, the start and end values of ``scipy.logspace`` must be
  entered as indices, e.g. to start the range at 0.1 and end it at 100 you
  would write ``scipy.logspace(-1, 2, steps)``. Setting up a PySCeS scan
  session might look something like::

    >>> import scipy 
    >>> mod.scan_in = 'x0'
    >>> mod.scan_out = ['J_R1','J_R6','s2_ss','s7_ss'] 
    >>> scan_range = scipy.linspace(0,100,11)

Before starting the parameter scan, it is important to check that all the
model attributes involved in the scan do actually exist. For example,
``mod.J_R1`` is created when ``mod.doState()`` is executed, likewise all
the elasticities (``mod.ecR_S``) and control coefficients (``mod.ccJ_R``)
are only created when the ``mod.doMca()`` method is called. If all the
attributes exist you can perform a parameter scan using the
``mod.Scan1(scan_range)`` method which takes your predefined scan range as
an argument::

  >>> mod.Scan1(scan_range)

  Scanning ... 
  11 (hybrd) The solution converged. 
  (hybrd) The solution converged ...

  done.

When the scan has been successfully completed, the results are stored in
the array (``mod.scan_res``) that has ``mod.scan_in`` as its first column
followed by columns that represent the data defined in ``mod.scan_out`` (if
invalid steady states are generated during the scan they are replaced by
NaN). Scan1 also reports the scan parameter values which generated the
invalid states.} . If one or more of your input or output parameters is not
a valid model attribute, it will be ignored. Once the parameter scan data
has been generated, the next step is to visualise it using the
``mod.Scan1Plot()`` method::

 >>> mod.Scan1Plot(plot=[], title=None, log=None, format='lines', filename=None)

- *plot* if empty mod.scan_out is used, otherwise any subset of mod.scan_out (default=[])
- *filename* the filename of the PNG file (default=None, no export)
- *title* the plot title (default=None)
- *log* if None a linear axis is assumed otherwise one of ['x','xy','xyz'] (default=None)
- *format* the backend dependent line format (default='lines')  or the *CommonStyle* 'lines' or 'points'.

Called without any arguments Scan1Plot plots all of *mod.sim_out* against *mod.sim_in*.

Two dimension parameter scans
-----------------------------

Two dimension parameter scans can also easily be generated using the ``mod.Scan2D``
method::

 >>> mod.Scan2D(p1, p2, output, log=False)

- *p1* is a list of [model parameter 1, start value, end value, points]
- *p2* is a list of [model parameter 2, start value, end value, points]
- *output* the steady-state variable e.g. 'J_R1' or 'A_ss'
- *log* if True scan using log ranges for both axes

To plot the results of two dimensional scan use the ``mod.Scan2DPlot`` method. Note
that as Matplotlib cannot produce 3D plots the GnuPlot interface must be active 
(see the section on plotting later on in this guide)::

 >>> mod.Scan2DPlot(title=None, log=None, format='lines', filename=None)

- *filename* the filename of the PNG file (default=None, no export)
- *title* the plot title (default=None)
- *log* if None a linear axis is assumed otherwise one of ['x','xy','xyz'] (default=None)
- *format* the backend dependent line format (default='lines')  or the *CommonStyle* 'lines' or 'points'.

Multi-dimension parameter scans
-------------------------------

This new PySCeS feature allows multi-dimensional parameter scanning. Any
combination of parameters is possible and can be added as *master*
parameters that change independently or *slave* parameters whose change is
coordinated with the previously defined parameter. Unlike ``mod.Scan1()``
this function is accessed via the ``pysces.Scanner`` class that is
instantiated with a loaded PySCeS model object::

  >>> sc1 = pysces.Scanner(mod) 
  >>> sc1.addScanParameter('x3', 1, 10, 11) 
  >>> sc1.addScanParameter('k2', 0.1, 1000, 5, log=True) 
  >>> sc1.addScanParameter('k4', 0.1, 1000, 5, log=True, slave=True)
  >>> sc1.addUserOutput('J_R1', 's1_ss') 
  >>> sc1.Run()

  ... scan: 55 states analysed

  >>> sc1_res = sc1.getResultMatrix()
  >>> print sc1_res[0]
  array([1., 0.1, 0.1, 97.94286647, 49.1380999])

  >>> print sc1_res[-1]
  array([1.0e+01, 1.0e+03, 1.0e+03, -3.32564878e+00, 3.84227702e-03])

In this scan we define two independent (``x3, k2``) and one dependent
(``k3``) scan parameters and track the changes in the steady state
variables ``J_R1`` and ``s1_ss``. Note that ``k2`` and ``k4`` use a
logarithmic scale. Once run the input parameters cannot be altered,
however, the output can be changed and the scan rerun.

* ``sc1.addScanParameter(name, start, end, points, log, slave)`` where
  ``name`` is the input parameter (as a string), ``start`` and ``end`` define
  the range with the required number of ``points``. While ``log`` and
  ``slave`` are boolean arguments indicating the point distribution and
  whether the axis is independent or not.

* ``sc1.addUserOutput(*args)`` an arbitrary number of model attributes to
  be output can be added (this method automatically tries to determine the
  level of analysis necessary) e.g. ``addUserOutput('J_R1', 'ecR1_k2')``

* ``sc1.Run()`` run the scan, if subsequent runs are required after
  changing output parameters use ``sc1.RunAgain()``. Note that it is not
  possible to change the input parameters once a scan has been run, if this
  is required a new Scanner object should be created.

* ``sc1.getResultMatrix(stst=False)`` return the scan results as an array containing
  both input and output if *stst = True* append the 
  steady-state fluxes and concentrations to the user output so 
  that output has dimensions [scan_parameters]+[state_species+state_flux]+[Useroutput] 
  otherwise return the default [scan_parameters]+[Useroutput].

* ``sc1.UserOutputList`` the list of output names

* ``sc1.UserOutputResults`` an array containing only the output

* ``sc1.ScanSpace`` the generated list of input parameters.

.. _Plotting:

Plotting
========

The PySCeS plotting interface has been completely rewritten to 
facilitate the use of multiple plotting back-ends via a Unified 
Plotting Interface (UPI). Using the UPI we ensure that a 
specified subset of plotting methods is back-end independent 
(although the UPI can be extended with back-end specific 
methods). So far Matplotlib (default) and GnuPlot back-ends 
have been implemented.

The common UPI functionality is accessible as ``pysces.plt.*`` 
while back-end specific functionality is available as 
``pysces.plt.m`` (Matplotlib) and ``pysces.plt.g`` (GnuPlot).

While the Matplotlib is activated by default GnuPlot needs to 
be enabled (see Configuring PySCeS section) and then activated 
using ``pysces.plt.p_activateInterface('gnuplot')``. All 
installed interfaces can be activated or deactivated as 
required:: 

 >>> pysces.plt.p_activateInterface(interface)
 >>> pysces.plt.p_deactivateInterface(interface)
  
where *interface* is either *'matplotlib'* or *'gnuplot'*. The 
PySCeS UPI defines currently has the following methods:

``plot(data, x, y, title='', format='')`` plot a single line data[y] vs data[x]

  - *data* the data array
  - *x* x column index
  - *y* y column index
  - *title* is the line key
  - *format* is the backend format string (default='')

``plotLines(data, x, y=[], titles=[], formats=[''])`` plot multiple lines i.e. data[y1, y2, ] vs data[x]
 
  - *data* the data array
  - *x* x column index
  - *y* is a list of line indexes, if empty all of y not including x is plotted
  - *titles* a list of line keys, if empty Line1, Line2, etc is used
  - *formats* a list (per line) of format strings, if formats only contains a single item, this format is used for all lines.

``splot(data, x, y, z, title='', format='')`` plot a surface i.e. data[z] vs data[y] vs data[x]

  - *data* the data array
  - *x* x column index
  - *y* y column index
  - *z* z column index
  - *title* the surface key
  - *format* a format string (default='')

``splotSurfaces(data, x, y, z=[], titles=[], formats=[''])`` plot multiple surfaces i.e. data[z1, z2, ] vs data[y] vs data[x]
 
  - *data* the data array
  - *x* x column index
  - *y* y column index
  - *z* a list of z column indexes, if empty all data not including x, y are plotted
  - *titles* a list of surface keys, if empty Surf1, Surf2 etc. is used
  - *formats* is a list (per line) of format strings (default='') 
 
If formats only contains a single item, this format is used for all surfaces.

``replot()`` replot the current figure using all active interfaces (useful with GnuPlot type interfaces)

``save(name, directory=None, dfmt='\%.8e')`` save the plot data and (if possible) the back-end specific format file

  - *filename* the filename
  - *directory* optional (default = current working directory)
  - *dfmt* the data format string (default='\%.8e')

``export(name, directory=None, type='png')`` export the current plot as a <format> file (currently only PNG is guaranteed to be available on all back-ends).
 
  - *filename* the filename
  - *directory* optional (default = current working directory)
  - *type* the file format (default='png').

``setGraphTitle(title='PySCeS Plot')`` set the graph title, unset if *title=None*
 
  - *title* (string, default='PySCeS Plot') the graph title

``setAxisLabel(axis, label='')`` sets one or more axis label
 
  - *axis* x, y, z, xy, xz, yz, zyx
  - *label* label string (default=None)
 
Called with only the axis argument clears that axis' label.

``setKey(value=False)`` enable or disable the current plot key, no arguments removes key.
 
  - *value* boolean (default = False)

``setLogScale(axis)`` set *axis* to log scale
 
  - *axis* is one of x, y, z, xy, xz, yz, zyx

``setNoLogScale(axis)`` set axis to a linear scale
 
  - *axis* is one of x, y, z, xy, xz, yz, zyx

``setRange(axis, min=None, max=None)`` set one or more axis range
 
  - *axis* is one of x, y, z, xy, xz, yz, zyx
  - *min* is the range(s) lower bound (default=None, back-end auto-scales)
  - *max* range(s) upper bound (default=None, back-end auto-scales)

``setGrid(value)`` enable or disable the graph grid
 
  - *value* (boolean) True (on) or False (off)

``plt.closeAll()`` Close all active Matplolib figures


.. _Output:

Displaying data
===============

Displaying/saving model attributes
----------------------------------

All of the ``showX()`` methods, with the exception of ``mod.showModel()``
operate in exactly the same way. If called without an argument, they
display the relevant information to the screen. Alternatively, if given an
open, writable (ASCII mode) file object as an argument, they write the
requested information to the open file. This allows the generation of
customised reports containing only information relevant to the model.

* ``mod.showSpecies()`` prints the current value of the model species
  (mod.M).

* ``mod.showSpeciesI()`` prints the initial, parsed in, value of the model
  species (mod.Mi).

* ``mod.showPar()`` prints the current value of the model parameters.

* ``mod.showState()`` prints the current steady-state fluxes and species.

* ``mod.showConserved()`` prints any moiety conserved relationships (if
  present).

* ``mod.showFluxRelationships()`` shows the relationships between dependent
  and independent fluxes at steady state

* ``mod.showRateEq()`` prints the reaction stoichiometry and rate equations.

* ``mod.showODE()`` prints the differential equations.

Please note that the ``mod.showModel()`` method is not 
recommended for saving models as a PySCeS input file instead 
use the Core2 based ``pysces.interface.writeMod2PSC`` method 
instead:: 

 >>> pysces.interface.writeMod2PSC(mod, filename, directory, iValues=True, getstrbuf=False)
 
- *filename*: writes <filename>.psc or <model_name>.psc if None
- *directory*: (optional) an output directory
- *iValues*: if True (default) then the models initial values are used (or the current values if False).
- *getstrbuf*: if True a StringIO buffer is returned instead of writing to disk

Assuming you have loaded a model and run ``mod.doState()`` the following
code opens a Python file object (``rFile``), writes the steady-state
results to the file associated with the file object (``results.txt``) and
then closes it again::

 >>> rFile = file('results.txt','w') 
 >>> mod.showState()      # print the results to screen
 >>> mod.showState(rFile) # write the results to the file results.txt
 >>> rFile.close()


Writing formatted arrays
------------------------

The ``showX()`` methods described in the previous sections allow the user a
convenient way to write the predefined matrices either to screen or file.
However, for maximum flexibility, PySCeS includes a suite of array writers
that enable one to easily write, in a variety of formats any array to a
file. Unlike the ``showX()`` methods, the ``Write_array`` methods are
specifically designed to write to data to a file.

In most modelling situations it is rare that an array needs to be stored or
displayed that does not have specific labels for its rows or columns.
Therefore, all the ``Write_array`` methods take list arguments that can
contain either the row or column labels. Obviously, these lists should be
equal in length to the matrix dimension they describe and in the correct
order.

There are currently three custom array writing methods that work either
with a 1D (vector) or 2D arrays (matrices). To allow an easy comparison of
the output of these methods, all the following sections use the same
example array as input.

``Write_array()``
~~~~~~~~~~~~~~~~~

The basic array writer is the ``Write_array()`` method. Using the default
settings this method writes a 'tab delimited' array to a file. It is
trivial to change this to a 'comma delimited' format by using the
``separator = ' '`` argument. Numbers in the array are formatted using the
global number format.

If column headings are supplied using the ``Col = []`` argument they are
written above the relevant column and if necessary truncated to fit the
column width. If a column name is truncated it is marked with a ``*`` and
the full length name is written as a comment after the array data.
Similarly row data can be supplied using the ``Row = []`` argument in which
case the row names are displayed as a comment which is written after the
array data.

Finally, if the ``close_file`` argument is enabled the supplied file object
is automatically closed after writing the array. The full call to the
method is::

  >>> mod.Write_array(input, File=None, Row=None, Col=None, separator=' ')

which generates the array

::

  ## Write_array_linear1_11:12:23 
  #s0           s1           s2 
  -3.0043e-001  0.0000e+000  0.0000e+000 
   1.5022e+000 -5.0217e-001  0.0000e+000 
   0.0000e+000  1.5065e+000 -5.0650e-001 
   0.0000e+000  0.0000e+000  1.0130e+000 
  # Row: R1 R2 R3 R4

By default, each time an array is written, PySCeS includes an array header
consisting of the model name and the time the array was written. This
behaviour can be disabled by setting: ``mod.write_array_header = 0``

``Write_array_latex()``
~~~~~~~~~~~~~~~~~~~~~~~~~

The ``Write_array_latex()`` method functions similarly to the generic
``Write_array()`` method except that it generates a formatted array that
can be included directly in a \LaTeX\ document. Additionally, there is no
separator argument, column headings are not truncated and row labels appear
to the left of the matrix.

::

  >>> mod.Write_array_latex(input, File=None, Row=None, Col=None)

which generates

::

  %% Write_array_latex_linear1_11:45:03 
  \[ 
  \begin{array}{r|rrr} 
    & $\small{s0}$ & $\small{s1}$ & $\small{s2}$ \\ \hline 
   $\small{R1}$ &-0.3004 & 0.0000 & 0.0000 \\ 
   $\small{R2}$ & 1.5022 &-0.5022 & 0.0000 \\
   $\small{R3}$ & 0.0000 & 1.5065 &-0.5065 \\ 
   $\small{R4}$ & 0.0000 & 0.0000 & 1.0130 \\ 
  \end{array} 
  \]

and in a typeset document appears as: 
  
  +---+--------+--------+--------+
  |   |     s0 |    s1  |     s2 |
  +---+--------+--------+--------+
  |R1 |-0.3004 | 0.0000 | 0.0000 |
  +---+--------+--------+--------+
  |R2 | 1.5022 |-0.5022 | 0.0000 |
  +---+--------+--------+--------+
  |R3 | 0.0000 | 1.5065 |-0.5065 | 
  +---+--------+--------+--------+
  |R4 | 0.0000 | 0.0000 | 1.0130 |
  +---+--------+--------+--------+
  
 
.. _Installing-PySCeS:

Installing and configuring
==========================

Before installing or building PySCeS the following software is 
required:

 - Python 2.5 (or 2.4 plus the Elementree/cElementree packages)
 - Numpy 1.2+
 - SciPy 0.7.0 (0.6.x will work with NumPy > 1.0.5)
 - GCC 4.2+ on Linux or MinGW GCC 3.4.5 on Windows is required for building PySCeS from source only
 - Matplotlib 0.98.3 with the TkAgg backend (this is the default, but optional, plotting package but can be replaced with GnuPlot)
 
optional, but highly recommended, packages:

 - libSBML 3.x install with the Python bindings for SBML support 
 - GnuPlot alternative plotting back-end
 - iPython highly recommended for interactive modelling sessions
 - SciTE editor for editing and running PySCeS based modelling programs

This software stack provides a powerful scientific programming 
platform which is used by PySCeS to provide a flexible Systems 
Biology Modelling environment. 

PySCeS 0.7.0 itself has been modularised into a main package 
and a (growing) number of support modules which extends its 
core functionality. The most important of these is the advanced 
simulation support added by installing PySundials 
(http://pysundials.sf.net). Linux users should build and 
install the SUNDIALS library and PySundials (build instructions 
on the PySundials web site). Windows users can simply download 
and install the *pysces_pysundials* module. 

 - *pysces_pysundials* a binary port of SUNDIALS+PySundials for Windows
 - *pysces_metatool* adds elementary mode support to PySCeS using MetaTool
 - *pysces_mariner* SOAP based web services gateway, including a PySCeS server and remote client
 - *pysces_kraken* (coming soon) PySCeS distributed processing module (currently distributed with PySCeS)
 
PySCeS and its extension modules use either the Python 
distutils or the Numpy distutils extensions. Assuming you have 
working versions of NumPy and SciPy on a Linux type operating 
systems building PySCeS is as easy as:: 

 python setup.py install
 
On Windows (with MinGW) depending on your system configuration 
this becomes:: 

 python setup.py config --compiler=mingw32 build --compiler=mingw32 install

In this release we have started prototyping Python egg support 
(currently only for windows) which is implemented via the 
*setupegg.py* build scripts. 

By default PySCeS installs with a version of ZIB's NLEQ2 
non-linear solver. This software is distributed under its own 
non-commercial licence. Please see the README.txt document provided 
with this PySCeS installation for details.  

Configuring
-----------

PySCeS has two configuration (\*.ini) files that allows one to 
specify global (per installation) and local (per user options). 
Currently the multiuser options are only fully realised on 
Linux based systems. Global options are stored in the 
*pyscfg.ini* file which is created in your PySCeS 
installation directory (this is a Windows version with the 
Linux defaults indicated with in \*value\*):: 

 [Pysces]
 install_dir = c:\python25\lib\site-packages\pysces
 gnuplot_dir = c:\model\gnuplot\binaries
 model_dir = os.path.join(os.getenv('HOMEDRIVE')+os.path.sep,'Pysces','psc')
 output_dir = os.path.join(os.getenv('HOMEDRIVE')+os.path.sep,'Pysces')
 *model_dir = os.path.join(os.path.expanduser('~'),'Pysces','psc')*
 *output_dir = os.path.join(os.path.expanduser('~'),'Pysces')*

The *[Pysces]* section contains information on the installation 
directory, the directory where the GnuPlot executable(s) can be 
found and the default model file and output directories. As we 
shall see some of these defaults can be overruled by the local 
configuration options:: 

 [ExternalModules]
 nleq2 = True

 [PyscesModules]
 pitcon = True

These sections define whether 3rd party algorithms (e.g. NLEQ2) 
are available for use, while the last section allows the alternate
plotting backends to be enabled or disabled::

 [PyscesConfig]
 gnuplot = True
 matplotlib = True

The user configuration files (pys_usercfg.ini) are created when 
PySCeS is imported/run for the *first time*. On Windows this is 
in ``<HOMEDRIVE>\Pysces`` while on Linux this is in 
``$HOME\Pysces``. Once created the user configuration files can 
be edited and will be used for every subsequent PySCeS session::

 [Pysces]
 output_dir = C:\mypysces
 model_dir = C:\mypysces\pscmodels
 gnuplot = False

Here I have customised my default model and output directories 
and disabled GnuPlot (enabled above). If required *gnuplot_dir* 
can also be set to point to an alternate location on a per user 
basis. Once you have PySCeS configured to your personal 
requirements you are ready to begin modelling. 


.. _References:


References
==========


.. rubric:: Footnotes

.. [#] Hofmeyr, J.-H.S. (2001) *Metabolic control analysis in a nutshell*, 
       in T.-M. Yi, M. Hucka, M. Morohashi, and H. Kitano, eds, Proceedings
       of the 2nd International Conference on Systems Biology, pp. 291-300.
       
.. [#] Kacser, H. and Burns, J. A. (1973), *The control of flux*,
       Symp. Soc. Exp. Biol. **32**, 65-104. 

.. [#] Heinrich and Rappoport (1974), *A linear steady-state treatment of 
       enzymatic chains: General properties, control and effector strength*, 
       Eur. J. Biochem. **42**, 89-95. 


.. _PySCeS:      http://pysces.sourceforge.net
.. _Python:      http://www.python.org
.. _Numpy:       http://numpy.sourceforge.net
.. _Scipy:       http://www.scipy.org
.. _Matplotlib:  http://matplotlib.sourceforge.net
.. _Ipython:     http://ipython.scipy.org
.. _wxPython:    http://www.wxpython.org
.. _Mingw:       http://www.mingw.org
.. _PLY:         http://systems.cs.uchicago.edu/ply
.. _MetaTool:    http://www.biologie.hu-berlin.de/biophysics/Theory/tpfeiffer/metatool.html
.. _ZIB:         http://www.zib.de
.. _HYBRD:       http://www.netlib.org
.. _NLEQ2:       http://www.zib.de/SciSoft/ANT/nleq2.en.html
.. _Cygwin:      http://www.cygwin.com
