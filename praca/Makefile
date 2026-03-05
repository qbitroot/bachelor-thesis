all: main-en.pdf

main-en.pdf: main-en.tex *.tex *.bib images/*
	pdflatex main-en
	bibtex main-en
	pdflatex main-en
	pdflatex main-en

clean:
	rm -f *.aux *.bbl *.blg *.lof *.log *.lot *.out *.toc

.PHONY: all clean
