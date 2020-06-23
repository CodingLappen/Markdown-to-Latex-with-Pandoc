CC := pdflatex
TARGETS := main
TEMP := $(addsuffix .tmp , $(TARGETS))
COPY := $(addsuffix .md , $(TEMP))
MAINS := $(addsuffix .tex, $(TEMP))
OBJ := $(patsubst %.md, %.tex,$(filter-out $(addsuffix .md,$(TARGETS)) ,$(wildcard *.md))) $(MAINS)
NOT_MAIN := $(filter-out $(MAINS), $(OBJ))


all: $(COPY) $(OBJ) $(TARGETS)

test: 
	@echo $(OBJ)

clean: 
	rm -f $(subst .tmp,,$(MAINS)).tex $(OBJ) *.snm *.toc *.out *.log *.nav *.aux $(COPY) *.pdf

$(COPY): %.tmp.md : %.md $(DEPS)
	cp $< $@
	$(foreach var ,  $(NOT_MAIN), echo "\input{$(var)}" >> $(COPY) ;)

$(NOT_MAIN): %.tex : %.md $(DEPS)
	pandoc $< --listings --slide-level 2 -t beamer -o $@ 
	
$(MAINS): %.tex : %.md $(DEPS)
	pandoc $< --template=default.beamer --highlight-style=zenburn --listings --slide-level 2 -o $(MAINS)

$(TARGETS):  $(MAINS) 
	mv $< $(subst .tmp,,$<)
	pdflatex -interaction nonstopmode $(subst .tmp,,$<)
	pdflatex -interaction nonstopmode $(subst .tmp,,$<)
