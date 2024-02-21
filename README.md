```typescript
export const useStore = (shouldListen = true): [StoreState, StoreDispatch] => {
	const setState = useState(state)[1];

	const dispatch: StoreDispatch = async ({ type, payload }) => {
		const newState = await updatedState(type, payload)
		state = { ...state, ...newState }

		for (const listener of listeners) listener(state)
	};

	const listenerFilter = useCallback((listener: React.Dispatch<React.SetStateAction<StoreState>>): boolean => (
		listener !== setState
	), [setState]);

	useEffect(() => {
		pipe(shouldListen, B.fold(constVoid, () => listeners.push(setState)));

		return () => {
			listeners = pipe(
				shouldListen,
				O.fromPredicate(I.of),
				O.map(() => pipe(listeners, A.filter(listenerFilter))),
				O.getOrElse(() => listeners),
			)
		}
	}, [setState, shouldListen]);

	return [state, useCallback(dispatch, [])];
}
```
