with tabela2 as (
select
sum(round((di.valor * di.quantidade - di.valor * di.quantidade * di.desconto / 100),1)) as total_t2
from documentos d
left outer join cliefornec c on c.codigo = d.cliente_codigo
left outer join documentos_historico dh on dh.documento = d.codigo and dh.codigo = 
	(select max(x.codigo) from documentos_historico x where x.documento = d.codigo )
left outer join documentos_status ds on ds.codigo = dh.status_codigo
inner join documentos_itens di on di.documento = d.codigo and di.tipo != 4
inner join produtos p on p.codigo = di.produto_codigo
where date_trunc('day', d.data_emissao) BETWEEN '01/01/2019' and '31/05/2019' and d.empresa_codigo = 1 and ds.codigo = 2 and di.valor !=0
),

tabela1 as (
SELECT
c.codigo,
c.nome,
t2.total_t2,
sum(round((di.valor * di.quantidade - di.valor * di.quantidade * di.desconto / 100),1)) as total_t1,
round(sum(round((di.valor * di.quantidade - di.valor * di.quantidade * di.desconto / 100),1)) / t2.total_t2 * 100,2) as perc
from documentos d
cross join tabela2 t2
left outer join cliefornec c on c.codigo = d.cliente_codigo
left outer join documentos_historico dh on dh.documento = d.codigo and dh.codigo = 
	(select max(x.codigo) from documentos_historico x where x.documento = d.codigo )
left outer join documentos_status ds on ds.codigo = dh.status_codigo
inner join documentos_itens di on di.documento = d.codigo and di.tipo != 4
inner join produtos p on p.codigo = di.produto_codigo
where date_trunc('day', d.data_emissao) BETWEEN '01/01/2019' and '31/05/2019' and d.empresa_codigo = 1 and ds.codigo = 2 and di.valor !=0
group by c.codigo, t2.total_t2
),

tabela3 as (
select
t1.*,
row_number() over (order by total_t1 DESC) as i
from tabela1 t1
)

select 
t3.*,
t3.perc + coalesce((select sum(coalesce(x.perc,0)) from tabela3 x where x.i < t3.i),0) as saldo,
(
	case 
    	when t3.perc + coalesce((select sum(coalesce(x.perc,0)) from tabela3 x where x.i < t3.i),0) <= 80 then 'A'
        when t3.perc + coalesce((select sum(coalesce(x.perc,0)) from tabela3 x where x.i < t3.i),0) BETWEEN 80.01 AND 95 then 'B'
		else 'C'
END
) as classe

from tabela3 t3
