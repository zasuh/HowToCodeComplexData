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

### April 11th 2018 ###
- Functions on mutually recursive data:
```
(require 2htdp/image)

;; fs-starter.rkt (type comments and examples)
;; fs-v1.rkt (complete data-definition plus function problems)

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
; .

(define F1 (make-elt "F1" 1 empty))
(define F2 (make-elt "F2" 2 empty))
(define F3 (make-elt "F3" 3 empty))
(define D4 (make-elt "D4" 0 (list F1 F2)))
(define D5 (make-elt "D5" 0 (list F3)))
(define D6 (make-elt "D6" 0 (list D4 D5)))
#;
(define (fn-for-element e)
  (... (elt-name e)    ;String
       (elt-data e)    ;Integer
       (fn-for-loe (elt-subs e))))
#;
(define (fn-for-loe loe)
  (cond [(empty? loe) (...)]
        [else
         (... (fn-for-element (first loe))
              (fn-for-loe (rest loe)))])) 


;; Functions:

; 
; PROBLEM
; 
; Design a function that consumes Element and produces the sum of all the file data in 
; the tree.
; 


;; Element -> Integer (MR)
;; ListOfElement -> Integer? (MR)
;; produce the sum of all the data in element (and its subs)
(check-expect (sum-data--element F1) 1)
(check-expect (sum-data--loe empty) 0)
(check-expect (sum-data--element D5) 3)
(check-expect (sum-data--element D4) (+ 1 2))
(check-expect (sum-data--element D6) (+ 1 2 3))

;(define (sum-data--element e) 0);stub
;(define (sum-data--loe loe) 0);stub

(define (sum-data--element e)
  (if (zero? (elt-data e))
      (sum-data--loe (elt-subs e))
      (elt-data e)))

(define (sum-data--loe loe)
  (cond [(empty? loe) 0]
        [else
         (+ (sum-data--element (first loe))
            (sum-data--loe (rest loe)))]))

; 
; PROBLEM
; 
; Design a function that consumes Element and produces a list of the names of all the elements in 
; the tree. 
; 


;; Element -> ListOfString
;; ListOfElement -> ListOfString
;; produce list of the names of all the elements in the tree
(check-expect (all-names--loe empty) empty)
(check-expect (all-names--element F1) (list "F1"))
(check-expect (all-names--element D5) (list "D5" "F3"))
(check-expect (all-names--element D4) (list "D4" "F1" "F2"))
(check-expect (all-names--loe (list D4 D5)) (append (list "D4" "F1" "F2") (list "D5" "F3")))
(check-expect (all-names--element D6) (list "D6" "D4" "F1" "F2" "D5" "F3"))

;(define (all-names--element e) empty) ;stub
;(define (all-names--loe loe) empty) ;stub

(define (all-names--element e)
  (cons (elt-name e)
        (all-names--loe (elt-subs e))))

(define (all-names--loe loe)
  (cond [(empty? loe) empty]
        [else
         (append (all-names--element (first loe))
              (all-names--loe (rest loe)))])) 
```

### April 12th 2018 ###
- Backtrack search on BST:
```

; 
; PROBLEM
; 
; Design a function that consumes String and Element and looks for a data element with the given 
; name. If it finds that element it produces the data, otherwise it produces false.
; 


;; String Element -> Integer or false
;; String ListOfElement -> Integer or false
;; search the given tree for an element with the given name, produce data if found; false otherwise
(check-expect (find--loe "F3" empty) false)
(check-expect (find--element "F3" F1) false)
(check-expect (find--element "F3" F3) 3)
(check-expect (find--element "D6" D6) 0)
(check-expect (find--element "F3" D4) false)
(check-expect (find--element "F1" D4) 1)
(check-expect (find--element "F2" D4) 2)
(check-expect (find--loe "F2" (cons F1 (cons F2 empty))) 2)
(check-expect (find--loe "F3" (cons F1 (cons F2 empty))) false) 
(check-expect (find--element "D4" D4) 0)
(check-expect (find--element "F3" D6) 3)
(check-expect (find--element "F1" D6) 1)

;(define (find--element n e) false) ;stub
;(define (find--loe n loe) false) ;stub


(define (find--element n e)
  (if (string=? (elt-name e) n)    
      (elt-data e)    
      (find--loe n (elt-subs e))))

(define (find--loe n loe)
  (cond [(empty? loe) false]
        [else
         (if (not (false? (find--element n (first loe)))) ;;is it found in (first loe)?
             (find--element n (first loe))
             (find--loe n (rest loe)))]))  ;;produce Integer or false depending on whether its found in (rest loe)
```

### April 13th 2018 ###
- Harry Potter family tree problem:
```
;; Data definitions:

(define-struct wiz (name wand patronus kids))
;; Wizard is (make-wiz String String String ListOfWizard)
;; interp. a wizard in a descendant family tree
;;         name is the first name
;;         wand is the wood their primary wand is made of ("" if unknown)
;;         patronus is a string  ("" if unknown)
;;         kids is their immediate children

;;
;; ListOfWizard is one of:
;;  - empty
;;  - (cons Wizard ListOfWizard)
;; interp. a list of wizards

(define ARTHUR
  (make-wiz "Arthur" "" "Weasel"
            (list (make-wiz "Bill" "" "" (list (make-wiz "Victoire"  "" "" empty)      
                                               (make-wiz "Dominique" "" "" empty)   
                                               (make-wiz "Louis"     "" "" empty)))   
                  (make-wiz "Charlie" "ash" "" empty)
                  (make-wiz "Percy" "" "" (list (make-wiz "Molly" "" "" empty)          
                                                (make-wiz "Lucy"  "" "" empty)))
                  (make-wiz "Fred"    ""    "" empty)
                  (make-wiz "George"  ""    "" (list (make-wiz "Fred" "" "" empty)     
                                                     (make-wiz "Roxanne"  "" "" empty)))
                  (make-wiz "Ron"     "ash" "Jack Russell Terrier" (list (make-wiz "Rose" "" "" empty)  
                                                                         (make-wiz "Hugo" "" "" empty)))
                  (make-wiz "Ginny"   ""    "horse" 
                            (list (make-wiz "James" "" "" empty)
                                  (make-wiz "Albus" "" "" empty)
                                  (make-wiz "Lily"  "" "" empty))))))
#;#;
(define (fn-for-wizard w)
  (... (wiz-name w)
       (wiz-wand w)
       (wiz-patronus w)
       (fn-for-low (wiz-kids w))))

(define (fn-for-low low)
  (cond [(empty? low) (...)]
        [else
         (... (fn-for-wizard (first low))
              (fn-for-low (rest low)))]))



;; ListOfPair is one of:
;;  - empty
;;  - (cons (list String String) ListOfPair)
;; interp. used to represent an arbitrary number of pairs of strings
(define LOP1 empty)
(define LOP2 (list (list "Harry" "stag") (list "Hermione" "otter")))
#;
(define (fn-for-lop lop)
  (cond [(empty? lop) (...)]
        [else
         (... (first (first lop))
              (second (first lop)) ;(first (rest (first lop)))
              (fn-for-lop (rest lop)))]))


;; ListOfString is one of:
;; - empty
;; - (cons String ListOfString)
;; interp. a list of strings
(define LOS1 empty)
(define LOS2 (list "a" "b"))

(define (fn-for-los los)
  (cond [(empty? los) ...]
        [else
         (... (first los)
              (fn-for-los (rest los)))]))

;; Functions

; PROBLEM 3:
; 
; Design a function that produces a pair list (i.e. list of two-element lists)
; of every person in the tree and his or her patronus. For example, assuming 
; that HARRY is a tree representing Harry Potter and that he has no children
; (even though we know he does) the result would be: (list (list "Harry" "Stag")).
; 
; You must use ARTHUR as one of your examples.
; 


;; Wizard -> ListOfPair
;; ListOfWizard -> ListOfPair
;; produce every wizard in tree together with their patronus
(check-expect (patroni--low empty) empty)
(check-expect (patroni--wiz (make-wiz "a" "b" "c" empty)) (list (list "a" "c")))
(check-expect (patroni--wiz ARTHUR)
              (list
               (list "Arthur" "Weasel")
               (list "Bill" "")
               (list "Victoire" "")
               (list "Dominique" "")
               (list "Louis" "")
               (list "Charlie" "")
               (list "Percy" "")
               (list "Molly" "")
               (list "Lucy" "")
               (list "Fred" "")
               (list "George" "")
               (list "Fred" "")
               (list "Roxanne" "")
               (list "Ron" "Jack Russell Terrier")
               (list "Rose" "")
               (list "Hugo" "")
               (list "Ginny" "horse")
               (list "James" "")
               (list "Albus" "")
               (list "Lily" "")))

;<templates taken from Wizard and ListOfWizard>

(define (patroni--wiz w)
  (cons (list (wiz-name w)
              (wiz-patronus w))
        (patroni--low (wiz-kids w))))

(define (patroni--low low)
  (cond [(empty? low) empty]
        [else
         (append (patroni--wiz (first low))
                 (patroni--low (rest low)))]))

; PROBLEM 4:
; 
; Design a function that produces the names of every person in a given tree 
; whose wands are made of a given material. 
; 
; You must use ARTHUR as one of your examples.
; 


;; Wizard String -> ListOfString
;; ListOfWizard String -> ListOfString
;; produce names of all descendants whose wand is made of given wood (including wiz)
(check-expect (has-wand-of-wood--low empty "x") empty)
(check-expect (has-wand-of-wood--wiz (make-wiz "a" "b" "c" empty) "x") empty)
(check-expect (has-wand-of-wood--wiz (make-wiz "a" "b" "c" empty) "b") (list "a"))                                           
(check-expect (has-wand-of-wood--wiz ARTHUR "ash") (list "Charlie" "Ron"))

;<templates taken from Wizard and ListOfWizard, with added atomic parameter>

(define (has-wand-of-wood--wiz w wood)
  (if (string=? (wiz-wand w) wood)
      (cons (wiz-name w)
            (has-wand-of-wood--low (wiz-kids w) wood))
      (has-wand-of-wood--low (wiz-kids w) wood)))

(define (has-wand-of-wood--low low wood)
  (cond [(empty? low) empty]
        [else
         (append (has-wand-of-wood--wiz (first low) wood)
                 (has-wand-of-wood--low (rest low) wood))]))
```

### April 14th 2018 ###
- Example for two One-of type function:
```

;; prefix-equal-starter.rkt

; PROBLEM: design a function that consumes two lists of strings and produces true
; if the first list is a prefix of the second. Prefix means that the elements of
; the first list match the elements of the second list 1 for 1, and the second list
; is at least as long as the first.
; 
; For reference, the ListOfString data definition is provided below.

;; =================
;; Data Definitions:

;; ListOfString is one of:
;; - empty
;; - (cons String ListOfString)
;; interp. a list of strings

(define LS0 empty)
(define LS1 (cons "a" empty))
(define LS2 (cons "a" (cons "b" empty)))
(define LS3 (cons "c" (cons "b" (cons "a" empty))))

#;
(define (fn-for-los los)
  (cond [(empty? los) (...)]
        [else 
         (... (first los)
              (fn-for-los (rest los)))]))

;; ==========
;; Functions:

;; ListOfString ListOfString -> Boolean
;; produce true if lsta is a prefix of lstb
(check-expect (prefix=? empty empty) true)
(check-expect (prefix=? (list "x") empty) false)
(check-expect (prefix=? empty (list "x")) true)
(check-expect (prefix=? (list "x") (list "x")) true)
(check-expect (prefix=? (list "x") (list "y")) false)
(check-expect (prefix=? (list "x" "y") (list "x" "y")) true)
(check-expect (prefix=? (list "x" "x") (list "x" "y")) false)
(check-expect (prefix=? (list "x") (list "x" "y")) true)
(check-expect (prefix=? (list "x" "y" "z") (list "x" "y")) false)

;(define (prefix=? lsta lstb) false) ;stub

(define (prefix=? lsta lstb)
  (cond [(empty? lsta) true]
        [(empty? lstb) false]
        [else (and (string=? (first lsta)                       
                             (first lstb))                      
                   (prefix=? (rest lsta) (rest lstb)))
                   ]))

#;
(define (prefix=? lsta lstb)
  (cond [(and (empty? lsta) (empty? lstb)) (...)]
        [(and (cons? lsta) (empty? lstb)) (... lsta ...)]
        [(and (cons? lstb) (empty? lsta)) (... lstb ...)]
        [(and (cons? lsta) (cons? lstb)) (... lsta lstb ...)]))
```

### April 15th 2018 ###
- Solution for 2 One-of P2: Merge
```
;; ListOfNumber ListOfNumber -> ListOfNumber
;; Consumes two lists and produces a combined list
(check-expect (merge empty empty) empty)
(check-expect (merge empty (list 1 2 3)) (list 1 2 3))
(check-expect (merge (list 1 2 3) empty) (list 1 2 3))
(check-expect (merge (list 1 2 3) (list 4 6 6)) (list 1 2 3 4 5 6))

;(define (merge l1 l2) empty) ;stub

(define (merge l1 l2)
  (cond [(empty? l1) l2]
        [(empty? l2) l1]
        [else
         (if (<= (first l1) (first l2))
             (cons (first l1)
                   (merge (rest l1) l2))
             (cons (first l2)
                   (merge l1 (rest l2))))]))
```

- Solution for 2 One-of P4: Pattern Match
```
;; Functions:

;; Pattern ListOflString -> Boolean
;; Produces true if the pattern matches the ListOflString
(check-expect (pattern-match? empty empty) empty)
(check-expect (pattern-match? empty (list "A")) true)
(check-expect (pattern-match? (list "B") empty) false)
(check-expect (pattern-match? (list "A" "N" "A") (list "x" "3" "y")) true)
(check-expect (pattern-match? (list "A" "N" "A") (list "l" "3" "y")) false)
(check-expect (pattern-match? (list "N" "A" "N") (list "l" "a" "4")) true)
(check-expect (pattern-match? (list "N" "A" "N") (list "l" "b" "c")) false)
(check-expect (pattern-match? (list "A" "N" "A" "N" "A" "N")
                              (list "V" "6" "T" "1" "Z" "4")) true)

;(define (pattern-match? pat lols) empty) ;stub

(define (pattern-match? pat lols)
  (cond [(empty? pat) true]
        [(empty? lols) false]
        [(string=? (first pat) "A")
         (and (alphabetic? (first lols))
              (pattern-match? (rest pat) (rest lols)))]
        [(string=? (first pat) "N")
         (and (numeric? (first lols))
              (pattern-match? (rest pat) (rest lols)))]))

;; 1String -> Boolean
;; produce true if 1s is alphabetic/numeric
(check-expect (alphabetic? " ") false)
(check-expect (alphabetic? "1") false)
(check-expect (alphabetic? "a") true)
(check-expect (numeric? " ") false)
(check-expect (numeric? "1") true)
(check-expect (numeric? "a") false)

(define (alphabetic? 1s) (char-alphabetic? (string-ref 1s 0)))
(define (numeric?    1s) (char-numeric?    (string-ref 1s 0)))
```

### April 16th 2018 ###
- Forming a local expression:
```
(local [(define a 1)  ; Local definitions
        (define b 2)] ; Valid only inside local
  (+ a b))            ; Body

(local [(define p "accio")
        (define (fetch n) (string-append p n))]
  (fetch "portkey"))
```
- Hoisting/lifting:
```
(define a 1)
(define b 2)

(+ a
   (local [(define b 3)]
     (+ a b))
   b) ;7

;;=================================================

(define b 1)

(+ b
   (local [(define b 2)]
     (* b b))
   b) ;6

(define b 2) ;hoisting
```

### April 18th 2018 ###