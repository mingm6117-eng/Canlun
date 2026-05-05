# TradingView ChanLun Core Indicator Notes

Do not paste this Markdown file into TradingView Pine Editor.
Paste only `chanlun_tradingview.pine`.

Source PDF: local scanned ChanLun manual provided by the user.

## OCR Pages Used

- PDF pages 73-83: trend center definition, center maintenance/break, and third-type buy/sell logic.
- PDF pages 90-104: center extension/new center, ZD/ZG/GG/DD definitions, and third-type buy/sell examples.
- PDF pages 106-113: buy/sell point completeness, first/second/third buy point relationships.
- PDF pages 478-484: inclusion processing, confirmed fractals, confirmed strokes, and line-segment prerequisites.
- PDF pages 487-491: line-segment characteristic sequence definitions. These are documented but not fully implemented in v1.

## Implemented Rules

- K-line inclusion is processed sequentially. In an upward context, a contained group merges to `[max low, max high]`; in a downward context, it merges to `[min low, min high]`.
- Fractals are detected only after inclusion processing. A strict top requires the middle normalized K-line to have both a higher high and higher low than its neighbors; a strict bottom mirrors that rule.
- Strokes are built from alternating confirmed top/bottom fractals. Same-type fractals replace the prior endpoint only when they are more extreme.
- A stroke center is created when three consecutive strokes have an overlapping price interval. The v1 center interval is maintained as the intersection of overlapping stroke ranges.
- Third buy/sell points are confirmed only after a stroke leaves the latest center and the following stroke retraces without re-entering the center.
- First and second buy/sell labels are deterministic approximations based on the nearest center, confirmed stroke pivots, and optional MACD divergence.

## Deliberate V1 Limits

- The script implements a stroke-center operating layer, not full recursive line-segment centers.
- Lesson 83 explicitly warns that using strokes to build the smallest center is less stable than using segments; v1 accepts that tradeoff for TradingView usability and exposes it through this note.
- First/second buy/sell detection is necessarily simplified because full ChanLun classification depends on completed multi-level trend types. Third buy/sell detection is the closest to the PDF's direct definition.
- The indicator waits for confirmed bars and confirmed fractals/strokes, so labels intentionally lag rather than repaint on incomplete bars.

## TradingView Constraints Reflected

- The script uses Pine Script v6.
- Drawing counts are bounded and old line/box/label objects are deleted through arrays.
- It is an `indicator`, not a `strategy`, so it provides visual structure and alerts but does not simulate orders.
