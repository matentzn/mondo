## THIS IS A BIOMAKE MAKEFILE ##

SRCM := ../ontology/mondo-edit.obo

#PATTERNS = hereditary acquired location_top location rare genetic dependence
PATTERNS = \
autosomal_dominant autosomal_recessive x_linked y_linked \
infantile congenital childhood adult\
chronic acute \
location location_top \
disease_by_dysfunctional_structure \
disease_series_by_gene \
basis_in_disruption_of_process disrupts_process \
inborn_metabolic \
environmental_stimulus \
autoimmune allergy allergic_form_of_disease autoimmune_inflammation inflammatory_disease_by_site \
infectious_disease_by_agent specific_infectious_disease_by_agent postinfectious_disease infectious_inflammation \
rare genetic acquired rare_genetic hereditary \
syndromic isolated \
malignant benign \
neoplasm neoplasm_by_origin \
cancer carcinoma carcinoma_in_situ sarcoma \
adenoma adenocarcinoma leiomyosarcoma lipoma lymphoma melanoma rhabdomyosarcoma small_cell_carcinoma squamous_cell_carcinoma \
neuroendocrine_neoplasm neuroendocrine_neoplasm_grade1

all: all_mondo_ld merged-ld.owl  all_disease  all_ordo all_ncit all_icd all_snomed

all_disease_ld: $(patsubst %,ld-disease-%.tsv,$(PATTERNS))
all_disease: $(patsubst %,lx-disease-%.tsv,$(PATTERNS))
all_ordo: $(patsubst %,lx-ordo-%.tsv,$(PATTERNS))
all_ncit: $(patsubst %,lx-ncit-%.tsv,$(PATTERNS))
all_icd: $(patsubst %,lx-icd10-%.tsv,$(PATTERNS))
all_snomed: $(patsubst %,lx-snomed_disorder-%.tsv,$(PATTERNS))
all_mondo_lx_tsv: $(patsubst %,lx-mondo-%.tsv,$(PATTERNS))
all_mondo_lx: $(patsubst %,lx-mondo-%.owl,$(PATTERNS))
all_mondo_lx_obo: $(patsubst %,lx-mondo-%-logical.obo,$(PATTERNS))
all_mondo_ld: $(patsubst %,ld-mondo-%.owl,$(PATTERNS))
all_mondo_ld_csv: $(patsubst %,%.csv,$(PATTERNS))
all_mondo_ld_obo: $(patsubst %,ld-mondo-%.obo,$(PATTERNS))

all_yaml: $(patsubst %,%.yaml,$(PATTERNS))

lx-mondo-$(P).tsv:
	blip-findall -r mondo_edit -u odputil -debug load -i disease_patterns.pro  -goal "load_onts($(P))" "new_tuple_from_lex($(P),C,V)" -select C-V -no_pred -use_tabs -label > $@.tmp && mv $@.tmp $@
lx-all-mondo-$(P).tsv:
	blip-findall -r mondo_edit -u odputil -debug load -i disease_patterns.pro  -goal "load_onts($(P))" "tuple_from_lex($(P),C,V)" -select C-V -no_pred -use_tabs -label > $@.tmp && mv $@.tmp $@

# lexical
lx-$(Ont)-$(P).tsv:
	blip-findall -r $(Ont) -goal "load_onts($(P))" -consult helper.pro -u odputil -i disease_patterns.pro "tuple_from_lex($(P),C,V),(disease_id(C);C='iri'),\+genus(C,_)" -select C-V -no_pred -use_tabs -label > $@.tmp && mv $@.tmp $@


%.yaml:
	blip-findall -u odputil -i disease_patterns.pro "write_yaml($*)" > $@.tmp && grep -v ^write_yaml $@.tmp > $@

#$(Ont)-$(P)-ld.tsv: $(P).yaml $(Ont).owl
#	dosdp-tools --prefixes=prefixes.yaml --ontology=$(Ont).owl --reasoner=elk --obo-prefixes=true --template=$< --outfile=$@.tmp query && ./util/fix-tsvs.pl $@.tmp > $@

#ld-$(Ont)-$(P).csv: 
#	blip-findall -r $(Ont) -r uberonp  -u odputil -i disease_patterns.pro -goal "load_onts($(P))" "write_tuple($(P))" -select C-V -no_pred -use_tabs -label > $@.tmp && mv $@.tmp $@

ld-mondo-%.tsv: %.yaml
	dosdp-tools --catalog=../ontology/catalog-v001.xml  --ontology=$(SRCM) --reasoner=elk --obo-prefixes=true --template=$< --outfile=$@.tmp query && ./fix-tsvs.pl $@.tmp > $@
.PRECIOUS: ld-mondo-%.tsv

roundtrip-mondo-%.owl: ld-mondo-%.tsv %.yaml
	dosdp-tools  --ontology=$(SRCM) --infile=$< --obo-prefixes=true --template=$*.yaml --outfile=$@.tmp generate && mv $@.tmp $@

%.csv: ld-mondo-%.tsv
	tsv2csv.py $< $@

#ld-%.tsv: ld-%.csv
#	csv2tsv.py $< $@


%-logical.owl: %.owl
	owltools $< --remove-annotation-assertions -o $@
%-logical.obo: %.owl
	owltools $< --remove-annotation-assertions $(SRCM) --add-imports-from-supports   -o -f obo $@.tmp && grep -v ^imports: $@.tmp > $@

# GENERIC
%.obo: %.owl
	owltools $< --merge-imports-closure --add-obo-shorthand-to-properties -o -f obo --no-check $@

lx-$(Ont)-$(P).owl: lx-$(Ont)-$(P).tsv 
	apply-pattern.py -n $(OBO)/$(Ont)/$(P).owl  -P curie_map.yaml -b http://purl.obolibrary.org/obo/ -i $< -p $(P).yaml  > $@.tmp && mv $@.tmp $@

ld-$(Ont)-$(P).owl: ld-$(Ont)-$(P).tsv 
	apply-pattern.py -S syns.json -n $(OBO)/$(Ont)/$(P).owl -P curie_map.yaml -b http://purl.obolibrary.org/obo/ -i $< -p $(P).yaml  > $@.tmp && mv $@.tmp $@
#	dosdp-tools  --ontology=$(SRCM) --reasoner=elk --obo-prefixes=true --template=$(P).yaml --outfile=$@.tmp generate && mv $@.tmp $@

merged-ld.owl: $(patsubst %,ld-mondo-%.owl,$(PATTERNS))
	owltools --create-ontology $(OBO)/mondo/dp.owl $^ --merge-support-ontologies -o $@

merged-%.obo: merged-%.owl
	owltools $< -o -f obo $@

# TODO: replace using dosdp-tools
syns.json:
	extract-obo-syns.pl -e $(HOME)/repos/ext.obo $(HOME)/repos/go-ontology/target/go.obo  > $@


# first mkdir tmp
patternize:
	pl2sparql  -e -c conf/main_conf.pl -A ../ontology/void.ttl -i mondo_edit -i imports doall
