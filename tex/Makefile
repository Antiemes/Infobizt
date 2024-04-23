TEX = xelatex -interaction nonstopmode
# -halt-on-error
#BIB = bibtex
DOCKERIMAGE = antiemes/latex


#MAINDOCUMENT = prez01 prez02
#BIBFILE = references.bib
#FIGURES = $(shell find *.eps *.png *.jpg)

#all: $(MAINDOCUMENT).tex $(MAINDOCUMENT).bbl $(FIGURES)

all: prez01.pdf prez02.pdf prez03.pdf prez04.pdf prez05.pdf prez06.pdf

%.pdf: %.tex
	$(TEX) $<
	$(TEX) $<

clean::
	rm -fv *.aux *.log *.bbl *.blg *.toc *.out *.lot *.lof *.loa $(MAINDOCUMENT).pdf

#$(MAINDOCUMENT).bbl: $(MAINDOCUMENT).tex $(BIBFILE)
#	$(TEX) $(MAINDOCUMENT)
#	$(BIB) $(MAINDOCUMENT)

docker-all::
	docker run --rm -v ${PWD}:/project -w /project $(DOCKERIMAGE)  make all
