# Sorting Visualizer - Improvement Specification

This document outlines the required changes and improvements for the Sorting Visualizer, based on the Researcher Agent's findings and user requests. This specification will guide the Coder Agent in implementing the necessary updates.

## 1. Critical Fixes

Address the following critical issues to improve stability and user experience:

*   **DOM Reset Clean-up:**
    *   **Problem:** `resetBars()` currently only removes CSS classes, but doesn't clear old DOM elements when a new array is generated, leading to flicker or duplicate bars on rapid successive clicks.
    *   **Action:** Modify `generateRandomArray()` to explicitly clear the `arrayContainer.innerHTML` *before* rendering the new array.
*   **UI Non-blocking During Long Sorts:**
    *   **Problem:** Long sorting operations (e.g., Bubble Sort with many elements) can block the main UI thread, making the page unresponsive.
    *   **Action:** Introduce a mechanism (e.g., `await new Promise(r => setTimeout(r, 0))` or `requestAnimationFrame`) within the inner loops of sorting algorithms (`bubbleSort`, `selectionSort`) to yield control back to the browser, allowing UI updates during the sort.
*   **Safe Height Calculation:**
    *   **Problem:** The height calculation `(value / maxValue) * 100 + 5` can result in `NaNpx` if `maxValue` is zero (e.g., an empty array or an array of all zeros), causing bars to disappear.
    *   **Action:** Add a guard in `renderArray()` to handle cases where `maxValue` is zero (or less than 5, given the min height of 5px) to prevent `NaNpx`. Ensure a sensible fallback height (e.g., 5px) or prevent rendering if `maxValue` is problematic.

## 2. Major Improvements

Implement the following to enhance functionality and user feedback:

*   **Separate Animation Delays:**
    *   **Problem:** A single `animationSpeedMs` constant governs all animation steps, which isn't ideal for differentiating between comparison and swap operations.
    *   **Action:** Introduce separate constants for comparison delay (`compareDelayMs`) and swap delay (`swapDelayMs`). Update `bubbleSort`, `selectionSort`, and `swap` functions to use these new constants appropriately.
*   **Correct Selection Sort Min-Index Handling:**
    *   **Problem:** The `swapping` class for the `minIndex` in `selectionSort` may persist incorrectly if the minimum element doesn't change position.
    *   **Action:** Ensure the `swapping` class is always removed from the `minIndex` after the inner loop completes for each outer iteration.
*   **UI Disabling While Sorting:**
    *   **Problem:** Control buttons (`generate-array-btn`, `bubble-sort-btn`, `selection-sort-btn`) remain clickable during a sort, potentially leading to overlapping operations and visual glitches.
    *   **Action:** Create a helper function, e.g., `toggleControls(disabled)`, that disables all control buttons at the start of any sort function and re-enables them upon completion.
*   **Consolidate Bar Dimension Constants via CSS Variables:**
    *   **Problem:** `max-width: 20px` in CSS and `min-height: 5px` logic in JS are magic numbers, making it hard to adjust visual scale.
    *   **Action:** Introduce CSS variables (e.g., `--bar-max-width`, `--bar-min-height`) in the `<style>` block. Update the `renderArray` function to use these CSS variables for `min-height` calculation (e.g., `getComputedStyle`) or adjust the height scaling to respect the CSS min-height. Ensure `max-width` is also consistently managed via CSS. For `--bar-min-height` ensure that the calculation in `renderArray` effectively uses this.
*   **Faster Default Speed:**
    *   **Problem:** The current default animation speeds (50ms compare, 100ms swap) can feel slow for larger arrays.
    *   **Action:** Adjust the initial `compareDelayMs` to `20ms` and `swapDelayMs` to `40ms` for a faster default experience.

## 3. Minor Refactorings

Apply these changes for better code quality and maintainability:

*   **Encapsulate Global State:**
    *   **Problem:** `array`, `numBars`, `animationSpeedMs` are global `let` variables, increasing the risk of pollution.
    *   **Action:** Encapsulate these variables within an immediately invoked function expression (IIFE) or an object to limit their scope.
*   **Remove Unused `numBars`:**
    *   **Problem:** `numBars` is only used to compute random array size and then becomes redundant.
    *   **Action:** Remove `numBars` as a separate variable after its initial use in `generateRandomArray()`. The `array.length` can serve the same purpose.
*   **Use Static Bar Collections:**
    *   **Problem:** `document.getElementsByClassName('array-bar')` returns a live `HTMLCollection`. While not critical here, it's generally safer to work with a static snapshot for iterative operations.
    *   **Action:** Convert `HTMLCollection` to an `Array` using `Array.from()` or the spread operator (`[...bars]`) when iterating over bars in sorting functions.
*   **Add ARIA Roles for Accessibility:**
    *   **Problem:** The array bars lack accessibility attributes.
    *   **Action:** In `renderArray()`, add `role="progressbar"` and `aria-valuenow` to each `array-bar` element. The `aria-valuenow` should reflect the current value of the bar (or its percentage height).
*   **Smooth Color Transitions for Swapping:**
    *   **Problem:** The `background-color` changes instantly when the `swapping` class is added/removed.
    *   **Action:** Add a `transition` property for `background-color` to the `.array-bar.swapping` CSS rule to make the color change smoother.

## 4. New Features/Enhancements

Implement the following new features:

*   **Speed Control Slider:**
    *   **Problem:** Users cannot adjust animation speed dynamically.
    *   **Action:** Add a UI element (`<input type="range">`) to the `controls` div. Label it clearly (e.g., "Animation Speed").
    *   **Details:**
        *   Min value: 1ms (fastest)
        *   Max value: 500ms (slowest)
        *   Default value: Should correspond to the `compareDelayMs` (20ms) and `swapDelayMs` (40ms) set in `Major Improvements`.
        *   Event Listener: Bind an `input` event listener to the slider that updates `state.compareDelayMs` and `state.swapDelayMs` dynamically. The `swapDelayMs` should always be double the `compareDelayMs` (or a similar proportional relationship).
        *   Display: Add a small `<span>` or `<output>` element to display the current speed value next to the slider.
*   **More Sorting Options:**
    *   **Problem:** Only Bubble Sort and Selection Sort are available.
    *   **Action:** Add at least two new sorting algorithm buttons (e.g., "Insertion Sort", "Quick Sort", "Merge Sort"). Implement the corresponding JavaScript functions (`insertionSort()`, `quickSort()`, `mergeSort()`, etc.) that follow the same async visualization pattern.
    *   **Details:**
        *   The new buttons should be added to the `controls` div.
        *   The new sorting functions should be integrated into the `runSort` mechanism.

## Implementation Details

*   **File to Modify:** `index.html` (specifically the `<style>`, HTML body, and `<script>` blocks).
*   **No new files.**
*   **All changes within the existing HTML structure.**
*   **Maintain existing visual style and dark theme.**
*   **Ensure `animationSpeedMs` is replaced by `compareDelayMs` and `swapDelayMs` in the relevant functions and calculations.**
*   **Test functionality after each major fix/improvement.**
