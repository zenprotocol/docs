==========
Cost Model
==========

.. highlight:: rst

--------
Notation
--------

.. list-table::
   :header-rows: 1

   * - Notation
     - Meaning
   * - :math:`\color{red}{c\left(\color{black}{M}\right)}`
     - Cost of :math:`M`
   * - :math:`\color{blue}{\left[ \color{black}{M}\right]}`
     - Modified (elaborated) term :math:`M`
   * - :math:`\underline{\color{red}{n}}`
     - The number :math:`\color{red}{n}` **as a term**

----
Cost
----

.. list-table::
   :header-rows: 1

   * -
     - Term
     - Cost
   * - Function Abstraction
     - :math:`\color{red}{c\left(\color{black}{ \lambda x.M }\right)}`
     - :math:`\color{red}{0}`
   * - Function Application
     - :math:`\color{red}{c\left(\color{black}{ M\:N }\right)}`
     - :math:`\color{red}{1 + c\left(\color{black}{M}\right) + c\left(\color{black}{N}\right)}`
   * - Conditional
     - :math:`\color{red}{c\left(\color{black}{ \textbf{if}\:M\:\textbf{then}\:N_{1}\:\textbf{else}\:N_{2} }\right)}`
     - :math:`\color{red}{3 + c\left(\color{black}{M}\right) + \max\left(c\left(\color{black}{N_{1}}\right),c\left(\color{black}{N_{2}}\right)\right)}`
   * - Pattern Matching
     - :math:`\color{red}{c\left(\color{black}{ \textbf{match}\:M\:\textbf{with}\:|\:P_{k}\rightarrow N_{k} }\right)}`
     - :math:`\color{red}{c\left(\color{black}{M}\right)+\sum_{k=1}^{n}c\left(\color{black}{P_{k}}\right)+\max\left\{ c\left(\color{black}{N_{k}}\right)\right\} _{k=1}^{n}}`
   * - Record Definition
     - :math:`\color{red}{c\left(\color{black}{ \left\{ F_{k}=M_{k};\right\} }\right)}`
     - :math:`\color{red}{n+\sum_{k=1}^{n}c\left(\color{black}{M_{k}}\right)}`
   * - Record Projection
     - :math:`\color{red}{c\left(\color{black}{ M.F }\right)}`
     - :math:`\color{red}{1 + c\left(\color{black}{M}\right)}`

-----------
Elaboration
-----------

.. list-table::
   :header-rows: 1

   * -
     - Term
     - Elaborated Term
   * - Function Abstraction
     - :math:`\color{blue}{\left[\color{black}{ \lambda x.M }\right]}`
     - :math:`\lambda x . \textbf{inc}\:\underline{\color{red}{c\left(\color{black}{M}\right)}}\:\text{(}\color{blue}{\left[ \color{black}{M}\right]}\text{)}`
   * - Function Application
     - :math:`\color{blue}{\left[\color{black}{ M\:N }\right]}`
     - :math:`\color{blue}{\left[ \color{black}{M}\right]}\:\color{blue}{\left[ \color{black}{N}\right]}`
   * - Conditional
     - :math:`\color{blue}{\left[\color{black}{ \textbf{if}\:M\:\textbf{then}\:N_{1}\:\textbf{else}\:N_{2} }\right]}`
     - :math:`\textbf{if}\:\color{blue}{\left[ \color{black}{M}\right]}\:\textbf{then}\:\color{blue}{\left[ \color{black}{N_{1}}\right]}\:\textbf{else}\:\color{blue}{\left[ \color{black}{N_{2}}\right]}`
   * - Pattern Matching
     - :math:`\color{blue}{\left[\color{black}{ \textbf{match}\:M\:\textbf{with}\:|\:P_{k}\rightarrow N_{k} }\right]}`
     - :math:`\textbf{match}\:\color{blue}{\left[ \color{black}{M}\right]}\:\textbf{with}\:|\:P_{k}\rightarrow \color{blue}{\left[ \color{black}{N_{k}}\right]}`
   * - Record Definition
     - :math:`\color{blue}{\left[\color{black}{ \left\{ F_{k}=M_{k};\right\} }\right]}`
     - :math:`\left\{ F_{k}=\color{blue}{\left[ \color{black}{M_{k}}\right]};\right\}`
   * - Record Projection
     - :math:`\color{blue}{\left[\color{black}{ M.F }\right]}`
     - :math:`\color{blue}{\left[ \color{black}{M}\right]}.F`
