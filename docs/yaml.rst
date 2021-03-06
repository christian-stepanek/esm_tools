.. highlight:: shell
.. The next sets up red text for commenting the document. DELETE before merging inito release
.. role:: red

================
YAML File Syntax
================

What Is YAML?
=============

`YAML` is a structured data format oriented to human-readability. Because of this property,
it is the chosen format for configuration and runscript files in `ESM-Tools` and the
recommended format for runscripts (though bash runscripts are still supported). These
`YAML` files are read by the `esm_parser` and then converted into a Python dictionary.
The functionality of the `YAML` files is further expanded through the `esm_parser` and
other `ESM-Tools` packages (i.e. calendar math through the `esm_calendar`). The
idea behind the implementation of the `YAML` format in `ESM-Tools` is that the user only
needs to create or edit easy-to-write `YAML` files to run a model or a coupled setup,
speeding up the configuration process, avoiding bugs and complex syntax.
The same should apply to developers that would like to implement their models
in `ESM-Tools`: the implementation consists on the configuration of a few `YAML` files.

.. warning::
   `Tabs` are not allowed as `yaml` indentation, and therefore, `ESM-Tools` will return an
   error every time a `yaml` file with `tabs` is invoked (e.g. `runscripts` and `config`
   files need to be `'tab-free'`).

YAML-Specific Syntax
~~~~~~~~~~~~~~~~~~~~

The main `YAML` **elements** relevant to `ESM-Tools` are:

  * **Scalars**: numbers, strings and booleans, defined by a `key` followed by ``:`` and a
    `value`, i.e.::

      model: fesom
      version: "2.0"
      time_step: 1800

  * **Lists**: a collection of elements defined by a `key` followed by ``:`` and an indented
    list of `elements` (numbers, strings or booleans) starting with ``-``, i.e.::

      namelists:
              - namelist.config
              - namelist.forcing
              - namelist.oce

   or a list of the same `elements` separated by ``,`` inside square brackets ``[elem1, elem2]``::

       namelists: [namelist.config, namelist.forcing, namelist.oce]

  * **Dictionaries**: a collection of `scalars`, `lists` or `dictionaries` nested inside a
    general `key`, i.e.::

      config_files:
              config:  config
              forcing: forcing
              ice:     ice

Some relevant **properties** of the ``YAML`` format are:

  * Only **white spaces** can be used for indentation. **Tabs are not allowed**.

  * Indentation can be used to structure information in as many levels as required, i.e. a dictionary
    ``choose_resolution`` that contains a list of dictionaries (``T63``, ``T31`` and ``T127``)::

      choose_resolution:
              T63:
                      levels: "L47"
                      time_step: 450
                      [ ... ]
              T31:
                      levels: "L19"
                      time_step: 450
                      [ ... ]
              T127:
                      levels: "L47"
                      time_step: 200
                      [ ... ]

  * This data can be easily imported as `Python` dictionaries, which is part of what the `esm_parser`
    does.

  * ``:`` should always be **followed** by a `white space`.

  * **Strings** can be written both **inside quotes** (``key: "string"`` or ``key: 'string'``) **or
    unquoted** (``key: string``).

  * `YAML` format is **case sensitive**.

  * It is possible to add **comments** to ``YAML`` files using ``#`` before the comment (same as in
    Python).

ESM-Tools Extended YAML Syntax
==============================

.. warning::
   Work in progress. This chapter might be incomplete. Red statements might be imprecise or not true.

`ESM-Tools` offers extended functionality of the `YAML` files through the
`esm_parser`. The following subsections list the extended `ESM-Tools`
syntax for `YAML` files including calendar and math operations (see
:ref:`yaml:Math and Calendar Operations`).
The :ref:`yaml:YAML Elements` section lists the `YAML` elements needed for configuration files and
runscripts.

Variable Calls
~~~~~~~~~~~~~~

Variables defined in a `YAML` file can be invoked on the same file or in oder files
provided that the file where it is defined is read for the given operation.
The syntax for calling an already defined variable is::

  "${name_of_the_variable}"

Variables can be nested in sections. To define a variable using the value of another one that is
nested on a section the following syntax is needed::

  "${<section>.<variable>}"

When using `esm_parser`, variables in components, setups, machine files, general information, etc.,
are grouped under sections of respective names (i.e. ``general``, ``ollie``, ``fesom``, ``awicm``, ...).
To access a variable from a different file than the one in which it is declared it is necessary to
reference the file name or label as it follows::

  "${<file_label>.<section>.<variable>}"

**Example**

Lets take as an example the variable ``ini_parent_exp_id`` inside the ``general`` section in the
`FESOM-REcoM` runscript ``runscripts/fesom-recom/fesom-recom-ollie-restart-daily.yaml``::

  general:
          setup_name: fesom-recom
          [ ... ]
          ini_parent_exp_id: restart_test
          ini_restart_dir: /work/ollie/mandresm/esm_yaml_test/${ini_parent_exp_id}/restart/
          [ ... ]

Here we use ``ini_parent_exp_id`` to define part of the restart path ``ini_restart_dir``. 
``general.ini_restart_dir`` is going to be called from the `FESOM-REcoM` configuration file
``configs/setups/fesom-recom/fesom-recom.yaml`` to define the restart directory for `FESOM`
``fesom.ini_restart_dir``::

  [ ... ]
  ini_restart_dir: "${general.ini_restart_dir}/fesom/"
  [ ... ]

Note that this line adds the subfolder ``/fesom/`` to the subdirectory.

If we would like to invoke from the same runscript some of the variables defined in another file,
for example the ``useMPI`` variable in ``configs/machines/ollie.yaml``, then we would need to use::

  a_new_variable: "${ollie.useMPI}"

Bare in mind that these examples will only work if both `FESOM` and `REcoM` are involved in the
`ESM-Tool` task triggered and if the task is run in `Ollie` (i.e. it will work for
``esm_runscripts fesom-recom-ollie-restart-daily.yaml -e <experiment_id> ...``).

ESM-Tools Variables
~~~~~~~~~~~~~~~~~~~

ESM-Tools provide a set of variables that can be called from `YAML` files without a previous
declaration:

.. warning::
   The following list contains entries that don't belong here (i.e. ``model_dir``). Review and correct.

.. csv-table::
   :header: Key, Description
   :widths: 15, 85

   start_date,          Model's start date.
   end_date,            Model's end date.
   initial_date,        :red:`I don't understand the diference between the start_date and initial_date and so on`
   final_date,          
   parent_date,         
   current_date,        Current date.
   next_date,           :red:`Following time step's date?`
   time_step,           Time step of the model.
   expid,               ID of the experiment.
   parent_expid,        Parent ID.
   esm_namelist_dir,    "Absolute path to the namelists folder (``<PATH>/esm_tools/namelists``)."
   esm_runscript_dir,   "Absolute path to the runscripts folder (``<PATH>/esm_tools/runscripts``)."
   model_dir,           Absolute path of the model directory (where it was installed by `esm_master`).

Switches (``choose\_``)
~~~~~~~~~~~~~~~~~~~~~~~

A `YAML` list named as ``choose_<variable>`` function as a `switch` that evaluates the given ``variable``.
The nested element `keys` inside the ``choose_<variable>`` act as `cases` for the switch and the `values` of
this elements are only defined outside of the ``choose_<variable>`` if they belong to the selected
``case_key``::

  variable_1: case_key_2

  choose_variable_1:
          case_key_1:
                  configuration_1: value
                  configuration_2: value
                  [ ... ]
          case_key_2:
                  configuration_1: value
                  configuration_2: value
                  [ ... ]
          "*":
                  configuration_1: value
                  configuration_2: value
                  [ ... ]

The key ``"*"`` or ``*`` works as an `else`.

**Example**

An example that can better illustrate this general description is the `FESOM 2.0` resolution
configuration in ``<PATH>/esm_tools/configs/fesom/fesom-2.0.yaml``::

  resolution: CORE2

  choose_resolution:
          CORE2:
                  nx: 126858
                  mesh_dir: "${pool_dir}/meshes/mesh_CORE2_final/"
                  nproc: 288
          GLOB:
                  nx: 830305

Here we are selecting the ``CORE2`` as default configuration set for the ``resolution`` variable,
but we could choose the ``GLOB`` configuration in another `YAML` file (i.e. a runscript), to override
this default choice.

In the case in which ``resolution: CORE2``, then ``nx``, ``mesh_dir`` and ``nproc`` will take the values
defined inside the ``choose_resolution`` for ``CORE2`` (``126858``, 
``runscripts/fesom-recom/fesom-recom-ollie-restart-daily.yaml``, and ``288`` respectively), once
resolved by the `esm_parser`, at the same **nesting level** of the ``choose_resolution``.

.. Note::
   ``choose_versions`` inside configuration files is treated in a special way by the `esm_master`. To
   avoid conflicts in case an additional ``choose_versions`` is needed, include the compilation information
   inside a ``compile_infos`` section (including the ``choose_versions`` switch containning compilation
   information). Outside of this exception, it is possible to use as many ``choose_<variable>`` repetitions
   as needed.

Append to an Existing List (``add_``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Given an existing list ``list1``::

  list1:
      - element1
      - element2

it is possible to add members to this list by using the following syntax::

  add_list1:
      - element3
      - element4

so that the variable ``list1`` at the end of the parting will contain
``[element1, element2, element3, element4]``. This is not only usefull when you need to build the list
piecewise (i.e. and expansion of a list inside a ``choose_`` switch) but also as the
:ref:`hierarchy:File Hierarchy` will cause repeated variables to be overwritten.

**Properties**

  * It is possible to have multiple ``add_`` for the same variable in the same or even in different
    files. That means that all the elements contained in the multiple ``add_`` will be added to the
    list after the parsing.

**Exceptions**

Exceptions to ``add_`` apply only to the environment and namelist ``_changes`` (see
:ref:`yaml:Environment and Namelist Changes (\`\`_changes\`\`)`). For variables of the type ``_changes``,
an ``add_`` is only needed if the same ``_changes`` block repeats inside the same file. Otherwise, the
``_changes`` block does no overwrite the same ``_changes`` block in other files, but their elements
are combined.

**Example**

In the configuration file for `ECHAM` (``configs/components/echam/echam.yaml``) the list
``input_files`` is declared as::

  [ ... ]

  input_files:
      "cldoptprops": "cldoptprops"
      "janspec": "janspec"
      "jansurf": "jansurf"
      "rrtmglw": "rrtmglw"
      "rrtmgsw": "rrtmgsw"
      "tslclim": "tslclim"
      "vgratclim": "vgratclim"
      "vltclim": "vltclim"

  [ ... ]

However different `ECHAM` scenarios require additional input files, for example the ``HIST`` scenario
needs a ``MAC-SP`` element to be added and we use the ``add_`` functionality to do that::

  [ ... ]
  choose_scenario:
      [ ... ]
      HIST:
          forcing_files:
              [ ... ]
          add_input_files:
              MAC-SP: MAC-SP
      [ ... ]

An example for the ``_changes`` **exception** can be also found in the same ``ECHAM`` configuration file.
Namelist changes necessary for `ECHAM` are defined inside this file as::

  [ ... ]

  namelist_changes:
      namelist.echam:
          runctl:
              out_expname: ${general.expid}
              dt_start:
                  - ${pseudo_start_date!year}
                  - ${pseudo_start_date!month}
                  [ ... ]

This changes specified here will be combined with changes in other files (i.e. ``echam.namelist_changes``
in the coupled setups `AWICM` or `AWIESM` configuration files), not overwritten. However, `ECHAM`'s
version ``6.3.05p2-concurrent_radiation`` needs of further namelist changes written down in the same
file inside a ``choose_`` block and for that we need to use the ``add_`` functionality::

  [ ... ]

  choose_version:
      [ ... ]
      6.3.05p2-concurrent_radiation:
          [ ... ]
          add_namelist_changes:
              namelist.echam:
                  runctl:
                      npromar: "${npromar}"
                  parctl:

  [ ... ]


Math and Calendar Operations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following math and calendar operations are supported in `YAML` files:

Arithmetic Operations
---------------------
An element of a `YAML` file can be defined as the result
of the addition, subtraction, multiplication or division of variables with the format::

  key: "$(( ${variable_1} operator ${variable_2} operator ... ${variable_n} ))"

The `esm_parser` supports calendar operations through `esm_calendar`. When performing calendar
operations, variables that are not given in date format need to be followed by their ``unit`` for
the resulting variable to be also in date format, i.e.::

  runtime: $(( ${end_date} - ${time_step}seconds ))

``time_step`` is a variable that is not given in date format, therefore, it is necessary to use
``seconds`` for ``runtime`` to be in date format. Another example is to subtract one day from
the variable ``end_date``::

  $(( ${end_date} - 1days ))

The units available are:

===================== ==================
Units supported by arithmetic operations
========================================
calendar units        | seconds
                      | minutes
                      | days
                      | months
                      | years
===================== ==================

Extraction of Date Components from a Date
-----------------------------------------
It is possible to extract date components from a `date variable`. The syntax for such an operation
is::

   "${variable!date_component}"

An example to extract the year from the ``initial_time`` variable::

  yearnew: "${initial_date!syear}"

If ``initial_date`` was 2001-01-01T00:00:00, then ``yearnew`` would be 2001.

The date components available are:

========= ======================================
Date components
================================================
ssecond   Second from a given date.
sminute   Minute from a given date.
shour     Hour from a given date.
sday      Day from a given date.
smonth    Month from a given date.
syear     Year from a given date.
sdoy      Day of the year, counting from Jan. 1.
========= ======================================

Changing Namelists
------------------

It is also possible to specify namelist changes to a particular section:


.. code-block:: yaml

    echam:
            namelist_changes:
                    namelist.echam:
                            runctl:
                                    l_orbvsop87: false
                            radctl:
                                    co2vmr: 217e-6
                                    ch4vmr: 540e-9
                                    n2ovmr: 245e-9
                                    cecc: 0.017
                                    cobld: 23.8
                                    clonp: -0.008
                                    yr_perp: "remove_from_namelist"

In the example above, the `namelist.echam` file is changed in two specific chapters, first the section ``runctrl`` parameter ``l_orbsvop87`` is set to ``.false.``, and appropriate gas values and orbital values are set in ``radctl``. Note that the special entry ``"remove_from_namelist`` is used to delete entries. This would translate the following fortran namelist (trucated)

.. code-block:: fortran

        &runctl
            l_orbvsop87 = .false.
        /
        
        &radctl
            co2vmr = 0.000217
            ch4vmr = 5.4e-07
            n2ovmr = 2.45e-07
            cecc = 0.017
            cobld = 23.8
            clonp = -0.008
        /



Globbing
~~~~~~~~

Globbing allows to use ``*`` as a wildcard in filenames for restart, input and output files.
With this feature files can be copied from/to the work directory whose filenames are not
completely known. The syntax needed is::

  file_list: common_pathname*common_pathname

Note that this also works together with the :ref:`yaml:List Loops`.

**Example**

The component `NEMO` produces one restart file per processor, and the part of the file name
relative to the processor is not known. In order to handle copying of restart files under
this circumstances, globbing is used in `NEMO`'s configuration file
(``configs/components/nemo/nemo.yaml``)::

  [ ... ]

  restart_in_sources:
      restart_in: ${expid}_${prevstep_formatted}_restart*_${start_date_m1!syear!smonth!sday}_*.nc
  restart_out_sources:
      restart_out: ${expid}_${newstep_formatted}_restart*_${end_date_m1!syear!smonth!sday}_*.nc

  [ ... ]

This will include inside the ``restart_in_sources`` and ``restart_out_sources`` lists, all the files
sharing the specified common name around the position of the ``*`` symbol, following the same rules
used by the Unix shell.

Environment and Namelist Changes (``_changes``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

List Loops
~~~~~~~~~~

This functionality allows for basic looping through a `YAML list`. The syntax for this is::

  "[[list_to_loop_through-->ELEMENT_OF_THE_LIST]]"

where ``ELEMENT_OF_THE_LIST`` can be used in the same line as a variable. This is
particularly useful to handle files which names contain common strings (i.e. `outdata` and
`restart` files, see :ref:`yaml:File Dictionaries`).

The following example uses the list loop functionality inside the ``fesom-2.0.yaml``
configuration file to specify which files need to be copied from the `work` directory
of runs into the general experiment `outdata` directory. The files to be copied for runs
modeling a couple of months in year 2001 are ``a_ice.fesom.2001.nc``, ``alpha.fesom.2001.nc``,
``atmice_x.fesom.2001.nc``, etc. The string ``.fesom.2001.nc`` is present in all files so we
can use the list loop functionality together with calendar operations (:ref:`yaml:Math and Calendar
Operations`) to have a cleaner and more generalized configure file. First, you need to declare the
list of unshared names::

  outputs: [a_ice,alpha,atmice_x, ... ]

Then, you need to declare the ``outdata_sources`` dictionary::

  outdata_sources:
        "[[outputs-->OUTPUT]]": OUTPUT.fesom.${start_date!syear}.nc

Here, ``"[[outputs-->OUTPUT]]":`` provides the `keys` for this dictionary as ``a_ice``, ``alpha``,
``atmice_x``, etc., and ``OUTPUT`` is later used in the `value` to construct the complete file name
(``a_ice.fesom.2001.nc``, ``alpha.fesom.2001.nc``, ``atmice_x.fesom.2001.nc``, etc.).

Finally, ``outdata_targets`` dictionary can be defined to give different names to `outdata` files
from different runs using `calendar operations`::

  outdata_targets:
        "[[outputs-->OUTPUT]]": OUTPUT.fesom.${start_date!syear!smonth}.${start_date!sday}.nc

The values for the `keys` ``a_ice``, ``alpha``, ``atmice_x``, ..., will be
``a_ice.fesom.200101.01.nc``, ``alpha.fesom.200101.01.nc``, ``atmice_x.fesom.200101.01.nc``, ...,
for a January run, and ``a_ice.fesom.200102.01.nc``, ``alpha.fesom.200102.01.nc``,
``atmice_x.fesom.200102.01.nc``, ..., for a February run.

File Dictionaries
~~~~~~~~~~~~~~~~~

File dictionaries are a special type of `YAML` elements that are useful to handle input, output,
forcing, logging, binary and restart files among others (see `File dictionary types` table),
and that are normally defined inside the `configuration files` of models. File dictionary's `keys`
are composed by a file dictionary ``type`` followed by ``_`` and an ``option``, and the `elements`
consist of a list of ``file_tags`` as `keys` with their respective ``file_paths`` as `values`::

  type_option:
        file_tag1: file_path1
        file_tag2: file_path2

The ``file_tags`` need to be consistent throughout the different ``options`` for files to be
correctly handled by ESM-Tools. Exceptionally, ``sources`` files can be tagged differently but
then the option ``files`` is required to link sources tags to general tags used by the other
options (see `File dictionary options` table below).

**File dictionary types**

.. csv-table::
   :header: Key, Description
   :widths: 15, 85

   analysis,            User's files for their own analysis tools (i.e. to be used in the pre-/postprocessing).
   bin,                 Binary files.
   config,              Configure sources.
   couple,              Coupling files.
   ignore,              Files to be ignored in the copying process.
   forcing,             Forcing files. An example is described at the end of this section.
   log,                 Log files.
   mon,                 Monitoring files.
   outdata,             "Output configuration files. A concise example is described in :ref:`yaml:List Loops`."
   restart_in,          "Restart files to be copied from the **experiment directory** into the **run directory** (see :ref:`esm_runscripts:Experiment Directory Structure`), during the beginning of the `computing phase` (e.g. to copy restart files from the previous step into the new run folder)."
   restart_out,         "Restart files to be copied from the **run directory** into the **experiment directory** (see :ref:`esm_runscripts:Experiment Directory Structure`), during the `tidy and resubmit phase` (e.g. to copy the output restart files from a finished run into the **experiment directory** for later use the next run)."
   viz,                 Files for the visualization tool.

**File dictionary options**

.. csv-table::
   :header: Key, Description
   :widths: 15, 85

   sources,             "Source file paths or source file names to be copied to the target path. **Without this option no files will be handled by ESM-Tools**. If ``targets`` option is not defined, the files are copied into the default `target` directory with the same name as in the `source` directory. In that case, if two files have the same name they are both renamed to end in the dates corresponding to their run (``file_name.extension_YYYYMMDD_YYYYMMDD``)."
   files,               "Links the general file tags (`key`) to the `source` elements defined in ``sources``. ``files`` **is optional**. If not present, all `source` files are copied to the `target` directory, and the `source tags` need to be the same as the ones in ``in_work`` and ``targets``. If present, only the `source` files included in ``files`` will be copied (see the `ECHAM` forcing files example below)."
   in_work,             "Files inside the `work` directory of a run (``<base_dir>/<experiment_name>/run_date1_date2/work``) to be transferred to the `target` directory. This files copy to the `target` path even if they are not included inside the ``files`` option. ``in_work`` **is optional**."
   targets,             "Paths and new names to be given to files transferred from the `sources` directory to the `target` directory. A concised example is described in :ref:`yaml:List Loops`. ``targets`` **is optional**."

File paths can be absolute, but most of the ``type_option`` combinations have a default folder
assigned, so that you can choose to specify only the file name. The default folders are:

.. csv-table::
   :header: Default folders, sources, in_work, targets
   :widths: 10, 30, 30, 30

   **bin**,             
   **config**,          
   **ignore**,          
   **forcing**,         
   **log**,             
   **outdata**,         ``<base_dir>/<experiment_name>/run_date1_date2/work``,          ``<base_dir>/<experiment_name>/run_date1_date2/work``,          ``<base_dir>/<experiment_name>/outdata/<model>``
   **restart_in**,      
   **restart_out**,     

**Example for ECHAM forcing files**

The `ECHAM` configuration file (``<PATH>/configs/echam/echam.yaml``) allows for choosing different
scenarios for a run. These scenarios depend on different combinations of forcing files. File sources
for all cases are first stored in ``echam.datasets.yaml`` (a ``further_reading`` file) as::

  forcing_sources:
        # sst
        "amipsst":
                "${forcing_dir}/amip/${resolution}_amipsst_@YEAR@.nc":
                        from: 1870
                        to: 2016
        "pisst": "${forcing_dir}/${resolution}${ocean_resolution}_piControl-LR_sst_1880-2379.nc"

        # sic
        "amipsic":
                "${forcing_dir}/amip/${resolution}_amipsic_@YEAR@.nc":
                        from: 1870
                        to: 2016
        "pisic": "${forcing_dir}/${resolution}${ocean_resolution}_piControl-LR_sic_1880-2379.nc"

        [ ... ]

Here ``forcing_sources`` store **all the sources** necessary for all `ECHAM` scenarios, and tag
them with source `keys` (``amipsst``, ``pisst``, ...). Then, it is possible to choose among
these source files inside the scenarios defined in ``echam.yaml`` using ``forcing_files``::

  choose_scenario:
        "PI-CTRL":
                forcing_files:
                        sst: pisst
                        sic: pisic
                        aerocoarse: piaerocoarse
                        aerofin: piaerofin
                        aerofarir: piaerofarir
                        ozone: piozone
        PALEO:
                forcing_files:
                        aerocoarse: piaerocoarse
                        aerofin: piaerofin
                        aerofarir: piaerofarir
                        ozone: piozone
        [ ... ]

This means that for a scenario ``PI-CTRL`` the files that are handled by ESM-Tools will be
**exclusively** the ones specified inside ``forcing_files``, defined in the
``forcing_sources`` as ``pisst``, ``pisic``, ``piaerocoarse``, ``piaerofin``, ``piaerofarir``
and ``piozone``, and they are tagged with new general `keys` (``sst``, ``sic``, ...) that
are common to all scenarios. The source files not included in ``forcing_files`` won't be
used.

YAML Elements
=============

The `esm_parser` is used to read the multiple types of `YAML` files contained in `ESM-Tools`
(i.e. model and coupling configuration files, machine configurations, runscripts, etc.). Each of
these `YAML` files can contain two type of `YAML` elements:

  * **Tool-specific elements**: `YAML-scalars`, `lists` or `dictionaries` that include instructions and
    information used by `ESM-Tools`. These elements are predefined inside the `esm_parser` or other
    packages inside `ESM-Tools` and are used to control the `ESM-Tools` functionality.

  * **User-defined elements**: `YAML-scalars`, `lists` of `dictionaries` that contain information
    defined by the user for later use as variables in the same `YAML` file or other `YAML` files.

The following subsections list and describe the **Tool-specific elements** used to operate `ESM-Tools`
from different files.

.. Note::
   Most of the **Tool-specific elements** can be defined in any file (i.e. `configuration file`,
   `runscript`, ...) and, if present in two files used by ESM-Tools at a time, the value is chosen
   depending on the ESM-Tools file priority/read order (:red:`reference here to that section`).
   Ideally, you would like to declare as many elements as possible inside the `configuration files`,
   to be used by default, and change them in the `runscripts` when necessary. However, it is ultimately
   up to the user where to setup the Tool-specific elements; the element classification in the following
   sections is just suggestion on how to organize ESM-Tools input.

Configuration Files
~~~~~~~~~~~~~~~~~~~

The following keys should/can be provided inside configuration files for models and coupled setups
(``<PATH>/esm_tools/configs/<model_or_setup>``):

.. csv-table::
   :header: Key, Description
   :widths: 15, 85

   model,               Name of the model.
   version,             Version of the model.
   repository,          Address of the model's repository.
                                destination: "fesom-1.4"
   metadata,            "List to incude descriptive information about the model (i.e. ``Authors``, ``Institute``, ``Publications``, etc.) used to produce the content of :ref:`Supported_Models:Supported Models`. This information should be organized in nested `keys` followed by the corresponding description. Nested `keys` do not receive a special treatment meaning that you can include here any kind of information about the model. Only the `Publications` `key` is treated in a particular way: it can consist of a single element or a `list`, in which each element contains a link to the publication inside ``<>`` (i.e. ``- Title, Authors, Journal, Year. <https://doi.org/...>``)."
   restart_rate,        
   restart_unit,        
   resolution,          "Name for the desired resolution configuration defined inside the ``choose_resolution`` list."
   pool_dir,            Absolute path of the pool directory.
   setup_dir,           Absolute path of the setup directory.
   bin_dir,             Absolute path of the binary folder containing the model binaries.
   namelist_dir,        Absolute path of the namelists directory for the model.
   namelists,           "List of namelist files required for the model, and contained in ``namelist_dir`` folder."
   executable,          Name of the model executable file.
   choose_resolution,   List of dictionaries containing different resolution configurations.
   namelist_changes,    
   choose_lresume,      
   coupling_fields,     List of coupling field dictionaries containing coupling field variables.
   grids,               List of grid dictionaries containing grid parameters.
   ":ref:`yaml:File dictionaries`",     "`YAML` dictionaries used to handle input, output, forcing, logging, binary and restart files."

Runscripts
~~~~~~~~~~

The following keys should be provided inside runscripts
(``<PATH>/esm_tools/runscripts/<model>/<runscript.yaml>``):

.. csv-table::
   :header: Key, Description
   :widths: 15, 85

