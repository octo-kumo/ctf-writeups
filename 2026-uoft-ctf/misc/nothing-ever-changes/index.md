---
created: 2026-01-11T23:10
updated: 2026-01-12T00:38
description: ML + Hash collision challenge
solves: 33
points: 131
title: Nothing Ever Changes
---

## challenge summary

We need to produce 10+10 images, where 10 of them are pixel equal to the reference numbers 0-9, the other 10 can have at most `budget[i]` pixels different from the first group, but should have the same MD5, the second group should also be classified as a different number by MNIST model attached.

## adversarial attack

Before we make MD5 collider, we need to actually have the two image files to collide, this is the ML part of this challenge, where we modify as few pixels as possible and make the AI predict the label we want.

Since the image is very small `28 * 28`, and the budget also very small, we can just do greedy brute force, where we pick the pixel that increase the label we want the most, and repeat.

## hash clash

Now that we have 2 completely different images and want to make them have the same MD5 (10 pairs), how?

At first I tried hashclash, but all it did was make my fans go brrr and disk IO skyrocket.

I tried https://github.com/corkami/collisions/blob/master/scripts/png.py, it was pretty much instant, MD5 were equal, however VS code failed to open them so I assumed the images were broken and didn't think much, I was going to sleep with hashclash running.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1768192036/20260111232715955.png/273070343403215cb2abd5bbcfcbc5d8.png)

However Julian asked for those images, and because of this, I noticed that my file explorer actually recognised those images as images.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1768192130/20260111232850803.png/bf2388c61eec407b9bc7fd1c189f4a76.png)

ngl if not for julian I would not have noticed this.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1768192176/20260111232936443.png/fc5e8d6c4df07b7817fea1eb4b1b4284.png)

The rest is simple, I ran the same image loading process on the images and found only `collision-crc*.png` to work, which is enough.

Hashclash is a scam.

```flag
uoftctf{d1d_y0u_kn0w_4_UofT_pr0f3550r_m4d3_th3_JSMA_p4p3r(https://doi.org/10.48550/arXiv.07528)???}
```

And the adversarial attack script.

```python
def targeted_greedy_l0_attack(
    pil_img: Image.Image,
    bundle,
    target_label: int,
    budget: int,
    device: str = "cpu",
    early_stop_when_success: bool = True,
) -> Tuple[Image.Image, bool, int]:
    """
    Greedy targeted L0 attack:
      - pil_img: input PIL L 28x28
      - bundle: ModelBundle
      - target_label: integer target class
      - budget: maximum number of pixels allowed to differ from original (L0 constraint)
    Returns: (modified_pil_image, success_bool, pixels_changed_count)
    """
    dev = torch.device(device)
    # original raw image tensor in [0,1], shape [1,1,28,28]
    orig_raw = pil_to_raw_tensor01(pil_img, dev)
    current_raw = orig_raw.clone()
    H, W = int(orig_raw.shape[2]), int(orig_raw.shape[3])

    # quick check if already classified as target
    if get_prediction_from_raw(current_raw, bundle) == target_label:
        return pil_img.copy(), True, 0

    # set of pixel coordinates that are already changed (counted towards L0)
    changed_mask = np.zeros((H, W), dtype=np.bool_)

    # Precompute candidate values in raw [0,1] domain to try: 0.0 and 1.0 (extremes)
    candidate_values = [0.0, 1.0]
    candidate_tensor_vals = [torch.tensor(v, device=dev, dtype=current_raw.dtype) for v in candidate_values]

    # Greedy loop
    for step in range(budget):
        best_increase = None
        best_coord = None
        best_value = None
        best_logits = None

        # current target logit baseline
        with torch.no_grad():
            baseline_logits = get_logits_from_raw(current_raw, bundle)  # shape [1,C]
            baseline_target_logit = float(baseline_logits[0, target_label].item())

        # Evaluate all pixel positions that are still eligible to change
        # (we allow re-setting already changed pixels to another value, but we count unique pixels only)
        for y in range(H):
            for x in range(W):
                # If this pixel already equals candidate value we would set, skip that candidate
                orig_val = float(orig_raw[0, 0, y, x].item())
                cur_val = float(current_raw[0, 0, y, x].item())
                for cand_idx, cand_val_t in enumerate(candidate_tensor_vals):
                    cand_val = float(candidate_values[cand_idx])
                    # If setting to same rounded uint8 value would not change pixel, skip
                    if int(round(cand_val * 255.0)) == int(round(orig_val * 255.0)) and not changed_mask[y, x]:
                        # setting to this value would not count as a change and is identical to original => skip
                        pass
                    # If current already equal to candidate, skip
                    if int(round(cand_val * 255.0)) == int(round(cur_val * 255.0)):
                        continue

                    # Create candidate by changing single pixel
                    cand_raw = current_raw.clone()
                    cand_raw[0, 0, y, x] = cand_val_t

                    # Evaluate logits (no grad needed here)
                    logits = get_logits_from_raw(cand_raw, bundle)
                    target_logit = float(logits[0, target_label].item())

                    increase = target_logit - baseline_target_logit

                    # Prefer candidates that immediately cause success
                    pred = int(torch.argmax(torch.softmax(logits, dim=1), dim=1)[0].item())
                    is_success = (pred == target_label)

                    # Choose candidate with maximum target logit (tie-break by success)
                    better = False
                    if best_increase is None:
                        better = True
                    else:
                        if is_success and not (best_logits is not None and int(torch.argmax(torch.softmax(best_logits, dim=1), dim=1)[0].item()) == target_label):
                            # prefer immediate success even if increase slightly smaller
                            better = True
                        elif (not is_success) and (best_logits is not None and int(torch.argmax(torch.softmax(best_logits, dim=1), dim=1)[0].item()) == target_label):
                            # existing best already yields success, prefer it
                            better = False
                        else:
                            # normal numeric comparison
                            if increase > best_increase:
                                better = True

                    if better:
                        best_increase = increase
                        best_coord = (y, x)
                        best_value = cand_val
                        best_logits = logits
                        if is_success:
                            break
                if best_logits is not None and int(torch.argmax(torch.softmax(best_logits, dim=1), dim=1)[0].item()) == target_label:
                    # we already found a successful single-pixel change in this scan
                    break
            if best_logits is not None and int(torch.argmax(torch.softmax(best_logits, dim=1), dim=1)[0].item()) == target_label:
                break

        # If no candidate found (shouldn't happen), break
        if best_coord is None:
            break

        # Commit the best change
        y_best, x_best = best_coord
        current_raw[0, 0, y_best, x_best] = torch.tensor(best_value, device=dev, dtype=current_raw.dtype)
        # Mark changed if different from original
        orig_val = int(round(float(orig_raw[0, 0, y_best, x_best].item()) * 255.0))
        new_val = int(round(best_value * 255.0))
        if orig_val != new_val:
            changed_mask[y_best, x_best] = True

        # Check success
        pred_now = get_prediction_from_raw(current_raw, bundle)
        if pred_now == target_label:
            # convert to PIL and return
            out_arr = raw_tensor01_to_uint8(current_raw)
            return uint8_to_pil(out_arr), True, int(changed_mask.sum())

    # finished budget or loop end, final check
    final_pred = get_prediction_from_raw(current_raw, bundle)
    out_arr = raw_tensor01_to_uint8(current_raw)
    success = (final_pred == target_label)
    return uint8_to_pil(out_arr), success, int(changed_mask.sum())
```
