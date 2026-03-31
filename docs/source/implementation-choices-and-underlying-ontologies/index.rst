Implementation choices and underlying ontologies
=================================================

**Reuse of existing ontologies and ontology terms**
^^^^^^^^^^^^^^
The NMD Domain Ontology embraces existing ontologies. By design, it prefers not to coin any new ontology terms itself, but solicits maintainers of existing ontologies to include new ontology terms for the concept from the NMD field not covered in existing ontologies yet.

**Design choices**
^^^^^^^^^^^^^^
- Use CARE-SM as a common data model (with possible mappings to OMOP).
- Ensure that the NMD ontology follows the CARE-SM<https://github.com/CARE-SM> data model logics. This means, for example, clear separation of a measurement process and the measurement that is output from that process
- Use existing ontology terms and definitions as much as possible. 
- Attach standard operating procedures for assessment and link these to the ontology terms. 

**Hierarchy of ontologies to be (partially) incorporated**
- Disease – primary ontologies MONDO; secondary ORDO, SNOMED, ICD10/11
- Phenotype – primary ontology HPO; secondary PhenX
- Clinical assessment – primary ontologies NCIT; secondary SNOMED
- Anatomy – primary ontology Uberon, secondary Uberon
- Function and capacity: ICF
- Assistive devices: ICF
- Therapies / (life style) intervention – medical action ontology

