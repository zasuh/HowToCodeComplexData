# Notes for How To Code: Complex Data #

### April 10th 2018 ###
- Mutually self-referential types can represent a tree of arbitrary depth and width.
- Where there is a self-reference (SR) there is usually a natural recursion (NR). That also means there is a natural mutual recursion (NMR).
- If there is a reference in the type comment, there is a natual helper *(NH)* in the template. 
- If there is a mutual reference cycle in the type comments, there is natual mutual recursion *(NMR)* in the templates. 
- If there is a self-reference type comments, there is natual recursion *(NR)* in the templates. 

```
(require 2htdp/image)

;; fs-starter.rkt (type comments and examples)

;; Data definitions:

(define-struct elt (name data subs))
;; Element is (make-elt String Integer ListOfElement)
;; interp. An element in the file system, with name, and EITHER data or subs.
;;         If data is 0, then subs is considered to be list of sub elements.
;;         If data is not 0, then subs is ignored.

;; ListOfElement is one of:
;;  - empty
;;  - (cons Element ListOfElement)
;; interp. A list of file system Elements

(define F1 (make-elt "F1" 1 empty))
(define F2 (make-elt "F2" 2 empty))
(define F3 (make-elt "F3" 3 empty))
(define D4 (make-elt "D4" 0 (list F1 F2)))
(define D5 (make-elt "D5" 0 (list F3)))
(define D6 (make-elt "D6" 0 (list D4 D5)))

(define (fn-for-element e)
  (... (elt-name e)                ;String
       (elt-data e)                ;Integer
       (fn-for-loe (elt-subs e)))) ;ListOfElement
  )

(define (fn-for-loe loe)
  (cond [(empty? loe) (...)]
        [else
         (... (fn-for-element (first loe)) ;Element
              (fn-for-loe (rest loe)))     ;ListOfElement
              ]))
```