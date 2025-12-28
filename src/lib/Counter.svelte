<script>
  let count = $state(0)
  const increment = () => {
    count += 1
  }
</script>

<button onclick={increment}>
  count is {count}
</button>

<style>
  button{
    width:100%;
    padding:10px 12px;
    border-radius:10px;
    border:none;
    background:linear-gradient(90deg,#0b1220,#111827);
    color:var(--accent,#60a5fa);
    font-weight:700;
    cursor:pointer;
    box-shadow:0 8px 20px rgba(2,6,23,0.6);
    transition:transform .14s ease,box-shadow .14s ease;
  }
  button:hover{transform:translateY(-2px);box-shadow:0 18px 36px rgba(2,6,23,0.55)}
</style>
