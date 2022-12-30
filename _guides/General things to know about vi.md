---
---

# (Greeklish) General things to know about vi

There are two modes on vi
   * **Escape mode** (pou eimaste mesa otan anoigei o vi kai prin patisoyme otidipote)
   * **Insert mode** (patame esc kai eimaste mesa sto keimeno opou mporoume kai na grafoume afou patisoume i or insert

**Oles oi entoles tou vi ginontai sto escape mode kai sto insert mode mporoume  mono na  grafoume**

## editor commands
   * `vi filename` - anoigoume to arxeio

   * `esc dd` - delete 1 line
   * `esc d3d` - delete 3 lines
   * `esc x` - delete 1 character
   * `esc 3x` - delete 3 characters

   * `shift+j` - anevazw grammi afou prota exw valei ton kersora sti grammi prooris mou
   * `esc $` - end of line
   * `esc :$` - last line of text
   * `esc 0` - start of line
   * `esc :0` - first line
   * `w` (word) - move a word forward ex `10w` - move 10 words forward
   * `b` - move a word back e.x. `10b` - move 10 words back
   * `esc o` - add a line below
   * `esc O` - add a line above
   * `esc u` - undo tin teleutaia kinisi (ws teleutaia kinisi orozetai to teleutaio escape)
   * `esc U` - undo all

   * `esc /keyword` - search e.x. `esc/test` - psaxnw ti leksi test mesa sto keimeno
   * `n` - next search

**An kanw lathos pataw esc kai pame pali apo tin arxi!**

   * `ctrl+l` - refresh tin othoni

   * `yy` - copy 1 line
   * `y10y` - copy 10 lines
   * `p` - paste (ekei pou tha valw ton kersora to paste tha ginei stin apo katw grammi)
   * `yw` - copy a word
   * `dw` - delete a word
   * `esc A` - Append ksekiname na grafoume sto telos tis grammis
   * `esc dG` - svise apo ton kersora ews to telos tou arxeiou

## escape commands `:`
   * `:wq` - save and quit
   * `:wq!` - save and quit oti kai na exei apo katw!!!
   * `:number of line` - se paei sti grammi pou tou eipes e.x. `esc:3` - se paei sti grammi 3
   * `:$` - last line of text
   * `:0` - first line
   * `:w` - save
   * `:w!` - save kai min mas ta prizeis - overwrite
   * `:w filename` - save as
   * `:e filename` - edit to arxeio
   * `:r filename` - sou episinaptei to arxeio sto simeio pou eixes ton kersora
   * `:start,end s/pattern/replace/` - search and replace e.x. `:1,$ s/joint/splif/g` (an thelw to replacement na ginei globally diladi se olo to keimeno)
