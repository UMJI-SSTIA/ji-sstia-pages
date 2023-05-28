all: beamer.tex diy.tex diy_solution.tex

*.tex:
	latexmk -xelatex $@

clean:
	latexmk -c *.tex
	rm -f *.nav *.snm *.vrb *.pre
