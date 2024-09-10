# Implementaion of 2024 CS50AI project "Minesweeper".

Initially I struggled to pass all the Check50 tests. Specifically:

    :( MinesweeperAI.add_knowledge can infer mine when given new information
    Cause
    expected "{(3, 4)}", not "set()"

    :( MinesweeperAI.add_knowledge can infer multiple mines when given new information
    Cause
    expected "{(1, 0), (1, 1...", not "set()"

    :( MinesweeperAI.add_knowledge combines multiple sentences to draw conclusions
    Cause
    did not find (1, 0) in mines when possible to conclude mine

The problem appeared to be that the knowledge base was not being fully updated after new information was gained.

When may new knowledge be gained?

* After making a move
* After making an inference - safe squares or mines might be found.

And also:

* After updating the list of sentences - as a new inference might be possible.

The list of sentences should be updated any time a new mine or safe square is discovered, because existing sentences might include the mine or safe square.

To capture new knowledge, we can call a function to update the list of mines and safes, **plus all sentences in the knowledge base** - not just the mines and safes as hinted in the course problem specifcation.

    def update_sentences_safes_and_mines(self):
     
        while True:
            changes = False

            # updates sentences
            for sentence in self.knowledge:
                for cell in sentence.known_safes():
                    if self.mark_safe(cell):
                        changes = True
                for cell in sentence.known_mines():
                    if self.mark_mine(cell):
                        changes = True

            # update safes
            for cell in self.safes:
                if self.mark_safe(cell):
                    changes = True

            # update mines
            for cell in self.mines:
                if self.mark_mine(cell):
                    changes = True

            # break from loop if no changes have been made
            if changes == False:
                break

Note the loop: it is possible that updating the sentences, safes and cells might unlock new knowledge. So we loop over this procedure until no new knowledge (changes) are made. Failing to loop here is likely why some of the Check50 tests were failing.

Note also that for this to work, the "mark" methods in the Sentence and self classes must return True if they make a change.

Here is a function to infer new knowledge:

       def infer(self):

        inferences = [
            Sentence(s2.cells - s1.cells, s2.count - s1.count)
            for s1 in self.knowledge
            for s2 in self.knowledge
            if s1.cells.issubset(s2.cells)
        ]
        return [inference for inference in inferences if inference not in self.knowledge]

It is possible that this new inferred knowledge can unlock further inferences, so we loop here too, calling the function to update the knowledge base with each iteration:

        while True:
            inferences = self.infer()
            if inferences:
                for inference in inferences:
                    self.knowledge.append(inference)
                self.update_sentences_safes_and_mines()
            else:
                # break from loop if no changes have been made
                break


Result:

    #5 submitted 28 minutes ago, Tuesday, September 10, 2024 4:37 AM +03
    check50 25/25 • style50 1.00 • 0 comments
    
Yay!


   


