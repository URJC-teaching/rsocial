# Uso rapido:
#   make                # compila el PDF
#   make SLIDES_TEX=otro.tex   # compila otro fichero .tex
#   make open           # abre el PDF generado
#   make clean          # limpia ficheros auxiliares
#   make distclean      # limpia auxiliares y PDF

SLIDES_TEX ?= slides_intro_auto.tex
MAIN := $(basename $(notdir $(SLIDES_TEX)))
OUTDIR ?= build

LATEXMK ?= latexmk
PDFLATEX ?= pdflatex

LATEX_FLAGS := -interaction=nonstopmode -halt-on-error -file-line-error

.PHONY: all intro tema_1 tema_2 tema_3 tema_4 tema_5 tema_6 pdf clean distclean rebuild open help

all: intro tema_1 tema_2 tema_3 tema_4 tema_5 tema_6

intro:
	$(MAKE) pdf SLIDES_TEX=intro.tex

tema_1:
	$(MAKE) pdf SLIDES_TEX=tema_1.tex

tema_2:
	$(MAKE) pdf SLIDES_TEX=tema_2.tex

tema_3:
	$(MAKE) pdf SLIDES_TEX=tema_3.tex

tema_4:
	$(MAKE) pdf SLIDES_TEX=tema_4.tex

tema_5:
	$(MAKE) pdf SLIDES_TEX=tema_5.tex

tema_6:
	$(MAKE) pdf SLIDES_TEX=tema_6.tex

pdf:
	@mkdir -p $(OUTDIR)
	$(LATEXMK) -pdf \
		-pdflatex="$(PDFLATEX) $(LATEX_FLAGS) %O %S" \
		-outdir=$(OUTDIR) $(SLIDES_TEX)
	@echo "PDF generado en: $(OUTDIR)/$(MAIN).pdf"

clean:
	$(LATEXMK) -c -outdir=$(OUTDIR) $(SLIDES_TEX)

# Limpieza completa incluyendo el PDF final
distclean:
	$(LATEXMK) -C -outdir=$(OUTDIR) $(SLIDES_TEX)
	rm -rf $(OUTDIR)

rebuild: distclean pdf

open: pdf
	xdg-open $(OUTDIR)/$(MAIN).pdf

help:
	@echo "Targets disponibles:"
	@echo "  make [pdf]            Compila el PDF"
	@echo "  make SLIDES_TEX=archivo.tex  Compila otro .tex"
	@echo "  make clean            Limpia auxiliares"
	@echo "  make distclean        Limpieza completa"
	@echo "  make rebuild          Recompila desde cero"
	@echo "  make open             Compila y abre el PDF"
