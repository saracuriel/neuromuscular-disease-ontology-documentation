=============================
Managing your ODK repository
=============================

Updating your ODK repository
----------------------------

Your ODK repositories configuration is managed in ``src/ontology/nmdo-odk.yaml``. The `ODK Project Configuration Schema <https://github.com/INCATools/ontology-development-kit/blob/master/docs/project-schema.md>`_ defines all possible parameters that can be used in this config YAML. Once you have made your changes, you can run the following to apply your changes to the repository:

.. code-block:: shell

   sh run.sh make update_repo

There are a large number of options that can be set to configure your ODK, but we will only discuss a few of them here.

.. note::

   **NOTE for Windows users:**  

   You may get a cryptic failure such as ``Set Illegal Option -`` if the update script located in ``src/scripts/update_repo.sh`` was saved using Windows Line endings. These need to change to Unix line endings. In Notepad++, for example, you can click on **Edit -> EOL Conversion -> Unix LF** to change this.

Managing imports
----------------

You can use the update repository workflow described above to perform the following operations on your imports:

1. Add a new import
2. Modify an existing import
3. Remove an import you no longer want
4. Customise an import

We will discuss all these workflows in the following sections.

Add new import
^^^^^^^^^^^^^^

To add a new import, first edit your ODK config as described above, adding an ``id`` to the ``product`` list in the ``import_group`` section (for the sake of this example, we assume you already import RO, and your goal is to also import GO):

.. code-block:: yaml

   import_group:
     products:
       - id: ro
       - id: go

.. note::

   Our ODK file should only have one ``import_group`` which can contain multiple imports in the ``products`` section. Next, run the update repo workflow to apply these changes. By default, this module is a SLME Bottom module (see `ROBOT extract <http://robot.obolibrary.org/extract>`_). To change that or customise your module, see the "Customise an import" section.

To finalize the addition of your import, perform the following steps:

1. Add an import statement to your ``src/ontology/nmdo-edit.owl`` file. You can copy an existing import declaration and rename it to the new ontology import:

   .. code-block:: text

       ...
       Ontology(<https://w3id.org/nmdo.owl>
       Import(<https://w3id.org/nmdo/imports/ro_import.owl>)
       Import(<https://w3id.org/nmdo/imports/go_import.owl>)
       ...

2. Add your imports redirect to your catalog file ``src/ontology/catalog-v001.xml``:

   .. code-block:: xml

       <uri name="http://purl.obolibrary.org/obo/nmdo/imports/go_import.owl" uri="imports/go_import.owl"/>

3. Test whether everything is in order:

   1. Refresh your import (see ``UpdateImports.md``)
   2. Open in your Ontology Editor of choice (Protege) and ensure that the expected terms are imported.

.. note::

   The catalog file ``src/ontology/catalog-v001.xml`` redirects imports from URLs to local files. Tools like ROBOT or Protégé use it to map e.g. 

   .. code-block:: text

       Import(<http://purl.obolibrary.org/obo/nmdo/imports/go_import.owl>)

   to 

   .. code-block:: xml

       <uri name="https://w3id.org/nmdo/imports/go_import.owl" uri="imports/go_import.owl"/>

Modify an existing import
^^^^^^^^^^^^^^^^^^^^^^^

If you simply wish to refresh your import in light of new terms, see ``UpdateImports.md``. To change the type of your module see "Customise an import".

Remove an existing import
^^^^^^^^^^^^^^^^^^^^^^^

To remove an existing import, perform the following steps:

1. Remove the import declaration from your ``src/ontology/nmdo-edit.owl``.
2. Remove the ``id`` from ``src/ontology/nmdo-odk.yaml`` (e.g., ``- id: go`` from the ``products`` list in ``import_group``).
3. Run the update repo workflow.
4. Delete the associated files manually:

   - ``src/imports/go_import.owl``
   - ``src/imports/go_terms.txt``

5. Remove the respective entry from ``src/ontology/catalog-v001.xml``.

Customise an import
^^^^^^^^^^^^^^^^^^

By default, an import module extracted from a source ontology will be a SLME module. To change it to a ROBOT filter module, modify your repo config (``src/ontology/nmdo-odk.yaml``):

.. code-block:: yaml

   import_group:
     products:
       - id: ro
       - id: go
         module_type: filter

.. note::

   A ROBOT filter module imports all external terms declared by your ontology. It does not consider terms/annotations outside the base namespace. You can add additional base IRIs as follows:

.. code-block:: yaml

   import_group:
     products:
       - id: go
         module_type: filter
         base_iris:
           - http://purl.obolibrary.org/obo/GO_
           - http://purl.obolibrary.org/obo/CL_
           - http://purl.obolibrary.org/obo/BFO

For a fully custom import, set ``module_type: custom`` and add a custom Makefile target:

.. code-block:: yaml

   import_group:
     products:
       - id: ro
       - id: go
         module_type: custom

.. code-block:: make

   imports/go_import.owl: mirror/ro.owl imports/ro_terms_combined.txt
   	if [ $(IMP) = true ]; then $(ROBOT) query  -i $< --update ../sparql/preprocess-module.ru \
   		extract -T imports/ro_terms_combined.txt --force true --individuals exclude --method BOT \
   		query --update ../sparql/inject-subset-declaration.ru --update ../sparql/postprocess-module.ru \
   		annotate --ontology-iri $(ONTBASE)/$@ $(ANNOTATE_ONTOLOGY_VERSION) --output $@.tmp.owl && mv $@.tmp.owl $@; fi

Add a component
^^^^^^^^^^^^^^

A component is an import which *belongs* to your ontology.  

1. Open ``src/ontology/nmdo-odk.yaml``
2. If you do not have it yet, add a new top-level section ``components``
3. Under ``components``, add a ``products`` section where all your components are specified
4. Add a new component, e.g., ``- filename: mycomp.owl``

.. code-block:: yaml

   components:
     products:
       - filename: mycomp.owl

When running ``sh run.sh make update_repo``, a new file ``src/ontology/components/mycomp.owl`` will be created, which you can edit manually or via:

- Using a ROBOT template
- Manual curation in Protégé or another editor
- Providing a ``components/mycomp.owl`` make target in ``src/ontology/nmdo.Makefile`` (custom command to generate component)  

  .. warning::

     Custom rules **must not** depend on other ODK-generated files such as seed files (see `issue <https://github.com/INCATools/ontology-development-kit/issues/637>`_).

- Specifying a ``source`` attribute in ``src/ontology/nmdo-odk.yaml`` to download the component from a URL.

Adding a new component based on a ROBOT template
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Open ``src/ontology/nmdo-odk.yaml``
2. Ensure ``use_templates: TRUE`` and ``use_context: TRUE`` in global project options. Add any non-standard prefixes to ``config/context.json``.
3. Add another component to the ``products`` section.
4. To activate template-driven behavior, set ``use_template: TRUE``. An empty template will be created in the templates directory.
5. To use multiple templates, add them under the ``templates`` field.
6. Advanced: Use ``template_options`` to provide extra options to ROBOT templates.

.. code-block:: yaml

   components:
     products:
       - filename: mycomp.owl
         use_template: TRUE
         template_options: --add-prefixes config/context.json
         templates:
           - template1.tsv
           - template2.tsv

.. note::

   For very large mirrors, see `ODK recommendation <https://github.com/INCATools/ontology-development-kit/blob/master/docs/DealWithLargeOntologies.md>`_.
