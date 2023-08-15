all:doc
.PHONY:doc

doc:report.tex
	pdflatex report.tex
	bibtex report.aux
	pdflatex report.tex
	pdflatex report.tex
	pdflatex report.tex
clean:
	rm -rf *.aux
	rm -rf *.dvi *.bbl *.blg *.log *.out *.toc *.nav *.snm *.loc *.soc
distclean:clean
	rm -f *.pdf
