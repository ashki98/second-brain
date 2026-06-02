# Confusion Matrix, L1, L2

Okay, I've adjusted the formulas in the summary to use the `$$...$$` block delimiters, which are generally well-supported in Notion for displaying mathematical equations using its KaTeX integration. I've also expanded the "Total" in the Accuracy formula for clarity.

Here is the revised summary with Notion-compatible formulas:

**Confusion Matrix (Classification Model Evaluation)**

- **Definition:** A table used to summarize the performance of a classification algorithm by comparing its predictions against the actual true values.
- **Purpose:** To understand not just the overall accuracy, but also the *types* of errors a model makes (where it gets "confused"). Essential for evaluating classification performance thoroughly.
- **Structure (Common Binary Case):** A 2x2 table:
    - Rows often represent **Actual** classes.
    - Columns often represent **Predicted** classes.
- **Key Components (Binary):**
    - **True Positive (TP):** Correctly predicted positive (Actual=Positive, Predicted=Positive).
    - **True Negative (TN):** Correctly predicted negative (Actual=Negative, Predicted=Negative).
    - **False Positive (FP):** Incorrectly predicted positive (Actual=Negative, Predicted=Positive) - *Type I Error*.
    - **False Negative (FN):** Incorrectly predicted negative (Actual=Positive, Predicted=Negative) - *Type II Error*.
- **Derived Performance Metrics:** Calculated *from* the confusion matrix counts:
    - **Accuracy:** Overall correctness.
    Accuracy=TP+TN+FP+FNTP+TN
    - **Precision:** Accuracy of positive predictions (Important when FP cost is high).
    Precision=TP+FPTP
    - **Recall (Sensitivity/True Positive Rate):** Ability to find all actual positives (Important when FN cost is high).
    Recall=TP+FNTP
    - **Specificity (True Negative Rate):** Ability to find all actual negatives.
    Specificity=TN+FPTN
    - **F1-Score:** Harmonic mean of Precision and Recall (Balances Precision & Recall).
    F1=2×Precision+RecallPrecision×Recall
- **Multi-Class:** Extends to an N x N matrix for N classes. Diagonal cells show correct classifications; off-diagonal cells show specific misclassifications between classes.

**L1 & L2 Regularization (Model Training Technique)**

- **Context:** Techniques applied during model *training* to improve generalization and prevent overfitting. They are *not* metrics derived from the confusion matrix itself.
- **Purpose:** To discourage overly complex models by adding a penalty to the loss function based on the magnitude of model weights/coefficients.
- **Mechanism:** Modify the loss function: New Loss=Original Loss+Penalty Term
- **L1 Regularization (Lasso):**
    - **Penalty:** Based on the sum of the *absolute values* of weights. The L1 norm:
    PenaltyL1=λi=1∑n∣wi∣=λ∣∣w∣∣1
    - **Effect:** Shrinks weights, potentially to *exactly zero*. Performs **feature selection**, resulting in sparse models.
- **L2 Regularization (Ridge):**
    - **Penalty:** Based on the sum of the *squared values* of weights. The squared L2 norm:
    PenaltyL2=λi=1∑nwi2=λ∣∣w∣∣22
    - **Effect:** Shrinks weights *towards* zero (rarely exactly zero). Reduces influence of less important features and handles correlated features well.
- **Hyperparameter $ \lambda $:** Controls the strength of the regularization penalty (higher $ \lambda $ = stronger penalty).
- **Relationship to Confusion Matrix:** **Indirect**. L1/L2 regularization affects *how the model is trained* and its final parameters. The confusion matrix is then used to *evaluate the performance* of the predictions made by this trained model.
- **Other Meanings:** L1/L2 can sometimes refer to Loss Functions (MAE/MSE) or Distance Norms (Manhattan/Euclidean), but in the context of model training/evaluation, it usually means Lasso/Ridge regularization.

You should be able to copy and paste this text into Notion, and the formulas within the `$$...$$` blocks should render correctly as mathematical equations.
