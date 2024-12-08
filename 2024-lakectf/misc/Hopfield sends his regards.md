---
created: 2024-12-07T18:23
updated: 2024-12-08T13:17
solves: 24
points: 325
---

The name is "Hopfield", that is a big give away that this is a [Hopfield network](https://en.wikipedia.org/wiki/Hopfield_network) question.

![hidden-in-the-noise.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733613933/2024/12/5db1bcbaa167a373868faffbca231369.png)

We are given an image and a `npz` file, loading the `npz` file gives us two numpy arrays, `W(1200, 1200)` and `v0(1200, )`.

The image itself is of size `120x10`, hence can be seen as a `(1200, )` data array.

Now let's run the network with the given weights against the given data inputs.

## running the network

```python
image = cv2.imread('hidden-in-the-noise.png')
image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
image = np.where(image < 255/2, 1, -1)
W = hopfield['W']
v0 = hopfield['v0']
def update_state(v, W):
	return np.sign(W @ v)
def decode(v):
	v = v.copy().flatten()
	max_iterations = 100
	for _ in range(max_iterations):
		new_v = update_state(v, W)
		if np.array_equal(new_v, v): # stablized
			break
		v = new_v
	return np.where(v == -1, 0, 255)
print("image decoded")
cv2_imshow(decode(image).reshape(10, 120).repeat(3, axis=0).repeat(3, axis=1))
print("image.T decoded")
cv2_imshow(decode(image.T).reshape(10, 120).repeat(3, axis=0).repeat(3, axis=1))
print("v0 decoded")
cv2_imshow(decode(v0).reshape(10, 120).repeat(3, axis=0).repeat(3, axis=1))
```

| var       | image                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `image`   | ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAWgAAAAeCAAAAAAkgtthAAACq0lEQVR4Ae2ZwW4cMQxDk2L//5dbbl78htUkQYPW6MVz8FAURXsVK1gkry/r+fny8rpw3gl5IAkbr/ynMgVayQQMK8MAT+KmghTq1kqdrZWhBLGktei7Cv8IeEypXJn5EcJT1Uo3+mHZAVs7cBq9tb2X+auT4owkCXmp/jdykD3bYJzQcXL4VKnnY/bKh2tm4FFOqIbyJmUCIuM5N3p1YvP7NHpzg5f9w+EK4z1f2X/57o2+6/vZweCHsyGANUpDdofpk+DWSpjWNCYbW54uDzOy50avPm1+n0ZvbvCyf37rYKYWc73HUHRISa8pQ8DI4GKJpg5UA2WDbCsd2LTD4K8Lx8FwaJ8WfJj1JCNLoYdRBtPZc6Pt0l5wGr23v7r/9q1DVpBZ8P47nmYbIBuzYwk+I0u5hW7UtgOjsSRZnAM6RTh2j6APQGGvVg2r8KYEfYa3/PMAkISs+p8b3W3ZiE+jNza3rR/c7VABee4T4eUfczFCZCHxeTP7YJTgEQdrrr7LQ35xHkosDLi7qcFWtz5n7yhWHxAxhWyRVR8wGlcFAT7nRtuKveA0em9/dX+AmJRggQqmjDWkQ4T4HrYSt9ZoK+iJu+8+GEN20RmekJT+DSJLdsjUw0ePLCugHRCH5yF0XfT11vPc6KspW9Fp9Nb2XubPXx1OykUv5M2/D5HjA2gl1fcSRuzDrOVkR7iO85x6thvm8E0iw4csOFakAuAxv+sHj0CZ4TCnKisCw3OjbcVecBq9t7+6v/9zNnHPEek7Y9nfg2+ZO57OY5g8HQYP0kO6lwKADh0idsUEc7Yz1TvicBeknNS50f449oLT6L391f39z6SOQxLcfxU7ANP0h84tbmx5H14yAHGPs3jImg+m0HIAJaaUCYbAEMG50d3zjfg0emNz2/oXRcrqSohovsEAAAAASUVORK5CYII=) |
| `image.T` | ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAWgAAAAeCAAAAAAkgtthAAACi0lEQVR4Ae2X0VbjQAxDy57+/y/vKrmtUD1J6MIxT87DVCPLmomxC3z8vd0+btsj4AemkI7+Dlhv9ZNz5cab5mum4SrIC4Cld7kOsaPlFKX/ycMG91VgCt1X2xfnOyMjLoG2TIHJl6RvbS6sFMqHoz2GGfpNnBcQ5uGqDsGLhNfqUGKR09HPEjZ/TqGbC/y0vwPo8ye5fTILZixYgTUXoLitSmZQvJUAH7emWOzc1NjH0WJ1LUgxGL3cBBw10NGEEhBFPx2dP51GPIVuLG5a32n4nC/CHoRUv4k1LH5wXt3WobNSueDU2NDAUQOHAOVQPFNzISBEimVs5XAGUBJl1d3QT0fvlehfptD9Nd5PePzVIcwAuvnZ0vmKmrdyTz/9/UvUKz7pyWRZkP5JGheAFWSxSqVtzzQXAodkSHoemqckzoPyfaejs0qNeArdWNy03r46mBHYMh05CBLkLKC3wAAZUa3wjhZ/y854CwBvypxlvYFDxbAI2OrayZ+9S5GR4kSypqNL5bu2U+iuyhbf7atDTc4DcM8/6c9Pz441K5Aa2WfajjDPKTvMhSwrnjhgu/pkCpqih8z1TIBVKg8N19ck0Tz+XHU6eq1nCzOFbinravr4h4X2Vpi29+wYrJn/y3iglLjaMmWHnk70JZF5a0FJT349UeIvBdY43YcqxJ1hzJs0z62mo6lD+zqFbi8xBzy+OtzweSykGc2Cn8Qm3wceNFIO3XQ6/OHdDhOd8v5NViUmXlPAtfO24JXPLGEJpqNLTbq2U+iuyhbfx1eHWPofoDVn52IkmZ0LQTnPWx9npgCcRRoUXPQlmlmrsojz/iRqFbnKdm5bMsW4AEwwnI526XrBFLq3vnb/B1xj3COVUs02AAAAAElFTkSuQmCC)                                             |
| `v0`      | ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAWgAAAAeCAAAAAAkgtthAAACq0lEQVR4Ae2ZwW4cMQxDk2L//5dbbl78htUkQYPW6MVz8FAURXsVK1gkry/r+fny8rpw3gl5IAkbr/ynMgVayQQMK8MAT+KmghTq1kqdrZWhBLGktei7Cv8IeEypXJn5EcJT1Uo3+mHZAVs7cBq9tb2X+auT4owkCXmp/jdykD3bYJzQcXL4VKnnY/bKh2tm4FFOqIbyJmUCIuM5N3p1YvP7NHpzg5f9w+EK4z1f2X/57o2+6/vZweCHsyGANUpDdofpk+DWSpjWNCYbW54uDzOy50avPm1+n0ZvbvCyf37rYKYWc73HUHRISa8pQ8DI4GKJpg5UA2WDbCsd2LTD4K8Lx8FwaJ8WfJj1JCNLoYdRBtPZc6Pt0l5wGr23v7r/9q1DVpBZ8P47nmYbIBuzYwk+I0u5hW7UtgOjsSRZnAM6RTh2j6APQGGvVg2r8KYEfYa3/PMAkISs+p8b3W3ZiE+jNza3rR/c7VABee4T4eUfczFCZCHxeTP7YJTgEQdrrr7LQ35xHkosDLi7qcFWtz5n7yhWHxAxhWyRVR8wGlcFAT7nRtuKveA0em9/dX+AmJRggQqmjDWkQ4T4HrYSt9ZoK+iJu+8+GEN20RmekJT+DSJLdsjUw0ePLCugHRCH5yF0XfT11vPc6KspW9Fp9Nb2XubPXx1OykUv5M2/D5HjA2gl1fcSRuzDrOVkR7iO85x6thvm8E0iw4csOFakAuAxv+sHj0CZ4TCnKisCw3OjbcVecBq9t7+6v/9zNnHPEek7Y9nfg2+ZO57OY5g8HQYP0kO6lwKADh0idsUEc7Yz1TvicBeknNS50f449oLT6L391f39z6SOQxLcfxU7ANP0h84tbmx5H14yAHGPs3jImg+m0HIAJaaUCYbAEMG50d3zjfg0emNz2/oXRcrqSohovsEAAAAASUVORK5CYII=) |

Running the network against `v0`, `image` does not give us any useful information.

## recovering patterns

We can recover patterns stored in a Hopfield network using analysing the eigenvectors and eigenvalues of the network's weight.

```python
Wn = W / np.sum(np.diag(W))
eval, evec = np.linalg.eigh(Wn)
idx = np.argsort(np.abs(eval))[::-1]
patterns = np.sign(evec[:, idx][:, :4])
cv2_imshow(np.vstack([np.where(patterns[:, i] == -1, 0, 255).reshape(10, 120) for i in range(4)]).repeat(3, axis=0).repeat(3, axis=1))
```

![](data:image/png;base64,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)

```flag
EPFL{a_n33dL3_in_teh_h@ystack}
```

This means the challenge is solvable even if we only have the weights matrix.
