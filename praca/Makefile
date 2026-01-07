# Choose one: main.pdf (Slovak) or main-en.pdf (English)
all: main.pdf

main.pdf: main.tex *.tex *.bib images/*
	pdflatex main
	bibtex main
	pdflatex main
	pdflatex main

main-en.pdf: main-en.tex *.tex *.bib images/*
	pdflatex main-en
	bibtex main-en
	pdflatex main-en
	pdflatex main-en

clean:
	rm -f *.aux *.bbl *.blg *.lof *.log *.lot *.out *.toc

.PHONY: all clean
