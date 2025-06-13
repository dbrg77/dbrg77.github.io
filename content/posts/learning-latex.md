---
title: "Learning Latex"
date: 2022-03-04T15:07:53+08:00
draft: true
type: post
---

Code block test

```tex
\documentclass[aspectratio=169]{beamer}
\usepackage[utf8]{inputenc}
% agjust makes \includegraphics takes the [center] option
\usepackage[export]{adjustbox}
\usepackage{amsmath,amssymb,amsthm,pgfplots,pgfplotstable}
% provide center, flushleft/right environments and commands \centering \justify
\usepackage{ragged2e}
% table related
\usepackage{booktabs,array,multirow}
% drawing and plotting
\usepackage{tikz}
\usepackage{setspace,xcolor}
% Use metropolis theme
\usetheme{metropolis}
%\usetheme[background=dark]{metropolis}
%\usecolortheme{metropolis-highcontrast}

% This light background makes my eyes more comfortable in a light environment
\definecolor{mybg}{RGB}{254,251,241}

% use normal Math font, instead of the metropolis one
\usefonttheme[onlymath]{serif}

\graphicspath{ {../figures/}{../../../} }
\usetikzlibrary{positioning, arrows.meta}
\usepgfplotslibrary{statistics}
\definecolor{darkred}{RGB}{176,35,24}
\definecolor{darkgreen}{RGB}{156,188,137}

% definte gauss function:
\pgfmathdeclarefunction{gauss}{2}{%
  \pgfmathparse{1/(#2*sqrt(2*pi))*exp(-((x-#1)^2)/(2*#2^2))}%
}

\begin{document}

\begin{frame}{i.i.d. Random Variables}
\vspace{10pt}
    \begin{columnsonlytextwidth}
        \column{0.7\textwidth}
        \begin{tikzpicture}
            \draw[opacity=0] (0,0) rectangle (10,7);
            \node (icm) at (3.5, 7) [anchor=north] {\includegraphics[scale=0.25]{ICM.png}};
            \draw node [right=1pt of icm, anchor=west] {$g \sim D(\mu, \sigma^2)$};
            % r.v.s
            \draw<2-> (0.5, 4.5) node (x1) {$X_1$};
            \draw<2-> (1.5, 4.5) node (x2) {$X_2$};
            \draw<2-> (2.5, 4.5) node (x3) {$X_3$};
            \draw<2-> (3.5, 4.5) node {$\cdots$};
            \draw<2-> (4.5, 4.5) node {$\cdots$};
            \draw<2-> (5.5, 4.5) node (x4) {$X_{n-1}$};
            \draw<2-> (6.5, 4.5) node (x5) {$X_{n}$};
            \path<2->[-, very thick, dashed, lightgray]
                (icm.south) edge (x1.north)
                (icm.south) edge (x2.north)
                (icm.south) edge (x3.north)
                (icm.south) edge (x4.north)
                (icm.south) edge (x5.north);
            % smal x
            \draw<2->[->, very thick, dashed, lightgray, >={Triangle[scale width=0.5]}]
                (x1.south) -- (0.5, 3.5) node [below] {\color{black} $x_1$};
            \draw<2->[->, very thick, dashed, lightgray, >={Triangle[scale width=0.5]}]
                (x2.south) -- (1.5, 3.5) node [below] {\color{black} $x_2$};
            \draw<2->[->, very thick, dashed, lightgray, >={Triangle[scale width=0.5]}]
                (x3.south) -- (2.5, 3.5) node [below] {\color{black} $x_3$};
            \draw<2->[->, very thick, dashed, lightgray, >={Triangle[scale width=0.5]}]
                (x4.south) -- (5.5, 3.5) node [below] {\color{black} $x_{n-1}$};
            \draw<2->[->, very thick, dashed, lightgray, >={Triangle[scale width=0.5]}]
                (x5.south) -- (6.5, 3.5) node [below] {\color{black} $x_n$};
            % x bar
            \draw<3-> (7.5,4.5) node[anchor=west, text width=10pt] {$\begin{aligned}
                \bar{X}=\frac{1}{n}\sum_{i=1}^{n}X_i
            \end{aligned}$};
            \draw<3->[->, very thick, dashed, lightgray, >={Triangle[scale width=0.5]}]
                (7.8, 4.25) -- (7.8,3.5) node[below] {\color{black} $\bar{x}$};
            % sampl 1
            \draw<4-> (0.5, 2.5) node {\color{blue} \footnotesize $2.3$};
            \draw<4-> (1.5, 2.5) node {\color{blue} \footnotesize $2.5$};
            \draw<4-> (2.5, 2.5) node {\color{blue} \footnotesize $3.3$};
            \draw<4-> (3.5, 2.5) node {\color{blue} \footnotesize $\cdots$};
            \draw<4-> (4.5, 2.5) node {\color{blue} \footnotesize $\cdots$};
            \draw<4-> (5.5, 2.5) node {\color{blue} \footnotesize $4.2$};
            \draw<4-> (6.5, 2.5) node {\color{blue} \footnotesize $2.9$};
            \draw<4-> (8.5, 2.5) node[anchor=west] {\color{blue} \footnotesize sample 1};
            % sample 1 mean
            \draw<5-> (7.8, 2.5) node {\color{blue} \footnotesize $3.0$};
            % sampl 2
            \draw<6-> (0.5, 1.75) node {\color{red} \footnotesize $4.3$};
            \draw<6-> (1.5, 1.75) node {\color{red} \footnotesize $1.5$};
            \draw<6-> (2.5, 1.75) node {\color{red} \footnotesize $3.8$};
            \draw<6-> (3.5, 1.75) node {\color{red} \footnotesize $\cdots$};
            \draw<6-> (4.5, 1.75) node {\color{red} \footnotesize $\cdots$};
            \draw<6-> (5.5, 1.75) node {\color{red} \footnotesize $3.0$};
            \draw<6-> (6.5, 1.75) node {\color{red} \footnotesize $2.5$};
            \draw<6-> (8.5, 1.75) node[anchor=west] {\color{red} \footnotesize sample 2};
            % sample 2 mean
            \draw<6-> (7.8, 1.75) node {\color{red} \footnotesize $3.2$};
            % sampl 3
            \draw<7-> (0.5, 1) node {\color{magenta} \footnotesize $2.3$};
            \draw<7-> (1.5, 1) node {\color{magenta} \footnotesize $1.8$};
            \draw<7-> (2.5, 1) node {\color{magenta} \footnotesize $5.8$};
            \draw<7-> (3.5, 1) node {\color{magenta} \footnotesize $\cdots$};
            \draw<7-> (4.5, 1) node {\color{magenta} \footnotesize $\cdots$};
            \draw<7-> (5.5, 1) node {\color{magenta} \footnotesize $2.0$};
            \draw<7-> (6.5, 1) node {\color{magenta} \footnotesize $4.5$};
            \draw<7-> (8.5, 1) node[anchor=west] {\color{magenta} \footnotesize sample 3};
            % sample 3 mean
            \draw<7-> (7.8, 1) node {\color{magenta} \footnotesize $2.8$};
            % omit
            \foreach \x in {1, 2, ..., 7}
            {
                \draw<7-> (\x-0.5, 0.4) node {\footnotesize $\vdots$};
            };
            \draw<7-> (7.8, 0.4) node {\footnotesize $\vdots$};
        \end{tikzpicture}

        \column{0.25\textwidth}<8->
        \justify
        \small $X_1, X_2, ...,X_n$ are independent and identically distributed (\textbf{i.i.d.}) random variables.\\[-20pt]
        \begin{align*}
            X_1 & \sim D(\mu, \sigma^2)\\
            X_2 & \sim D(\mu, \sigma^2)\\
            X_3 & \sim D(\mu, \sigma^2)\\
            &\vdots\\
            X_{n-1} & \sim D(\mu, \sigma^2)\\
            X_n & \sim D(\mu, \sigma^2)\\[10pt]
            \bar{X} & \sim\ ?(?,?)
        \end{align*}
    \end{columnsonlytextwidth}
\end{frame}

\end{document}
```

https://tex.stackexchange.com/questions/204697/how-to-correctly-typeset-an-authors-two-word-last-name-in-bibtex